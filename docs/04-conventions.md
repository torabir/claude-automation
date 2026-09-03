# 04 — Conventions

The unglamorous part that makes the rest work daily. These conventions exist because agentic workflows fail in predictable ways: context bloat, stale status, rules that live in one person's head.

## `progress.md` — the session handover

Every project carries the same docs skeleton:

```
docs/
├── spec.md        # requirements — what this thing is
├── roadmap.md     # milestones
├── todo.md        # the active milestone only
├── progress.md    # SESSION HANDOVER — read this first
└── decisions.md   # decision log (dated, append-only)
```

`progress.md` has one job: **give correct current status without reading code.** Each work session ends by updating it; each new session starts by reading it. This is what makes "new session per work session" viable — the handover is a file, not a 100k-token conversation history.

The decision log matters more than it looks: when an agent proposes re-litigating something ("should we use X instead?"), a dated entry saying *why not* ends the discussion in one line.

## Token economics as written policy

LLM context costs are managed like any other resource, with rules in the workspace `CLAUDE.md`:

- **New session per work session / per project.** Long context is re-sent with every message — the single biggest cost driver. Yesterday's mega-session is not continued; `progress.md` is the handover.
- **Batch small adjustments** into one message instead of five round-trips.
- **Still images before render iterations** — review mockups before paying for renders.
- **Ask before spawning parallel research subagents** (each costs on the order of a full context).
- **Validate inputs before LLM calls** — a failed call on an oversized transcript costs full price.

## Anatomy of a `CLAUDE.md` that actually works

Every project's `CLAUDE.md` follows the same shape, most binding first:

1. **Hard rules** — the things that must never happen (no auto-send, no voice cloning, ownership boundaries)
2. **Structure** — what lives where, in one screen
3. **Commands** — how to run the thing
4. **Conventions & pitfalls** — the mistakes already made once, so they're not made twice
5. **"Requires user confirmation"** — an explicit list: pushing, publishing, anything costing money beyond small test calls, deleting projects

That last section is the highest-value few lines in the file. It converts fuzzy judgment calls into a lookup table, for both the agent and the human. A reusable skeleton is in [`examples/CLAUDE.md.template`](../examples/CLAUDE.md.template).

Documentation discipline is enforced in the same spirit: the media pipeline's rule is that any change to CLI commands, approval gates or project file names **must** update the README runbook table in the same commit. Docs that can drift, will.

## The maturity ladder

Automation is promoted stepwise, never skipped ahead:

```
manual prompt  →  skill  →  automation
```

- Start as a **manual prompt**. If you find yourself giving the same instruction repeatedly, promote it to a **skill** (a versioned, reviewable instruction file).
- Promote a skill to unattended **automation** only when the workflow is stable *and* safe to run without eyes on it.
- Some categories are capped: **outbound actions (sending, publishing) never climb the ladder.** They stay human-gated forever, by policy.

This keeps the system honest about what's actually automated versus what's an agent being supervised — and it means every automated step went through a phase where a human watched it run many times.

## Patterns that recur across every pipeline

Having built several pipelines on these conventions (long-form video, short-form video, outreach drafting, application tooling), the same architecture keeps winning:

- **File-per-unit state** (one directory per video/opportunity/application) with zod-validated JSON manifests — greppable, diffable, no database to operate.
- **A state machine mutated through one module** — no stage writes status on its own.
- **LLM behind a provider interface**, called as a `claude -p` subprocess — no SDK dependency, no API key handling, swappable.
- **Deterministic logic in code, judgment in the LLM.** Scoring/assembly/validation are code; the LLM supplies sub-scores, rationales and drafts. When the LLM fails, the pipeline fails loudly instead of guessing.
- **Two-language separation:** working language with the human is Norwegian; code, CLI surfaces and commit history are English. Trivial to state, valuable to standardize.
