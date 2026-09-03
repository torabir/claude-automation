# 03 — Safety & security

Two problems, two mechanisms. **Security:** the agent must not be able to read secrets, even by accident. **Safety:** the agent must stop before irreversible actions. Both are enforced by configuration and code — not by asking the model nicely.

## Secrets: two layers, one rule list

The threat model is mundane: an agent with filesystem access will eventually read a `.env` while debugging, and then the key is in the conversation context. A `Read(.env)` deny rule alone is not enough — a shell command or a Python/Node subprocess can read the file through a side door.

So the workspace uses Claude Code's two enforcement layers together:

1. **Permission deny rules** (`permissions.deny`) block the built-in Read tool and known shell file-reads.
2. **The OS sandbox** (`sandbox.enabled: true` — Seatbelt on macOS) blocks *arbitrary subprocesses* from reading the same paths, because deny rules are merged into the sandbox's filesystem policy automatically.

One rule list, both layers. Verified empirically with dummy files: the Read tool is refused, and `cat .env` inside the sandbox returns `Operation not permitted` — the content never reaches the model. A pleasant side effect: the agent can't even *delete* a `.env`, its own test dummies included.

The full template is in [`examples/settings.security.json`](../examples/settings.security.json). Two design choices worth noting:

- **Enumerated variants instead of a blanket `.env.*`** — `.env.local`, `.env.production`, etc. are denied, but `.env.example` stays readable *on purpose*: it holds variable names only (never values) and the agent keeps it up to date when new variables appear. The tradeoff is covered by a naming convention (below).
- **Global + per-project.** Relative deny patterns only bind inside the workspace they're defined in. The same rules live in the user-level `~/.claude/settings.json` with absolute-path patterns (`//**/.env`), so every repo on the machine is covered — including ones created next year.

### The convention that makes the deny list sufficient

> Secrets live **only** in files named `.env` or inside a `secrets/` directory. Never in code, never in chat, never under other names.

An enumerated deny list can't block a key pasted into `notes.txt`. The convention is written into the workspace `CLAUDE.md`, so every session carries it.

### Rotation, not deletion

Also written into the rules: if a key ever lands in the wrong place, **rotate it at the provider** (create new → update `.env` and the host's env vars → verify → revoke old). Deleting the file doesn't un-leak the key. When this system was set up, every key the agent could plausibly have seen before the rules existed was rotated on that principle — and each rotation was verified with a minimal API call made by a script that reads `.env` itself and prints only OK/FAIL, so the new keys never entered the conversation either.

One operational detail people miss: gitignored secrets never reach production via git. Each hosting platform holds its own copy (e.g. env vars in the deploy platform's dashboard), so a rotation isn't done until *both* the local file and the platform's vault are updated.

## Safety: stops before the irreversible step

Every tool in the workspace shares one spine — the agent prepares, the human commits:

| Tool | The irreversible step | The mechanism |
|---|---|---|
| Media pipeline | Spending money (TTS), publishing | `blockers()` gate function in core; upload is manual, always |
| Outreach/application drafting | Sending anything | Status can only become "sent" with an explicit `--confirm --via "..."` flag attesting a *human* sent it manually |
| Job-application tooling | Submitting | Renders PDFs and cheat sheets; submission is human-only. Browser automation of form-filling was evaluated and **rejected** as net-negative |
| Personal assistant (calendar/email) *(in build)* | Sending email | **Guarantee by absence**: the OAuth scope technically allows sending, so the guarantee is that *no send path exists in the codebase at all*. All writes become proposal files requiring an explicit confirm command |
| Daily assistant | Pushing personal data | Day files live in a local-only git repo; commit yes, remote never |

"Guarantee by absence" is the strongest of these: instead of trusting a flag or a prompt to prevent sending, the capability is simply not implemented. You can audit it with `grep`.

Mass operations get the same treatment — `--all` style flags are explicitly forbidden in the tools' rules. Nothing here can be irreversible *at scale*, because nothing is irreversible even once.

## Governance for imported automation

Third-party skills are an injection surface: a SKILL.md is instructions the agent will follow. Standing rules in the workspace:

- No skill is imported without a **file-by-file audit**, logged in the project's decision log.
- `curl | sh` / `npx` recipes inside imported skills are never executed without per-instance approval.
- Marketplace MCP servers and "guru" tool bundles are not installed at all; the capability is rebuilt minimally in-house instead.

## What this does *not* claim

This is a solo-developer setup hardened against accidents and casual injection, not a zero-trust enterprise deployment. The sandbox has a documented, permission-gated escape hatch (running a specific command unsandboxed prompts the human), kept deliberately: pipelines occasionally need it, and a visible prompt beats a silent workaround. Defense in depth, honestly labeled.
