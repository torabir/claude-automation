# 01 — The assistant layer

The workspace root has a `CLAUDE.md` that turns Claude Code into a **router**: plain-language phrases trigger skills or route into projects, and everything else is just answered normally. No commands to memorize, no ritual forced on you.

## Natural-language triggers

| You say | What happens |
|---|---|
| "good morning" (or any morning greeting) | The **daily-brief skill** runs immediately, without asking |
| "good evening" / "evening check" | The **evening-review skill** runs |
| "let's work on \<project\>" | The **project router**: read that project's `CLAUDE.md`, `docs/progress.md` and active `docs/todo.md`; summarize status in 2–3 lines ("here's where we are, this is the next step"); continue under the project's own rules |
| anything else | Answered directly. The rule is explicit: *never force the ritual* — "how did X work again?" gets an answer, not a morning routine |

The routing table lives in `CLAUDE.md` as a short markdown section. That's the whole mechanism — no plugins, no framework. The router works because Claude Code loads the workspace `CLAUDE.md` into every session.

## The daily-brief skill

Six steps, every one skippable, hard-capped at ~5 minutes (a real design constraint — a morning routine that takes 20 minutes stops being used):

1. A short reflection nudge (content genericized here — make it yours)
2. A reminder to journal *outside* the tool (see privacy note below)
3. **Empty the head**: read yesterday's day file for unchecked `- [ ]` carry-overs plus a standing daily checklist; write today's `days/YYYY-MM-DD.md`
4. **Cross-project brief**: for each project, read `docs/progress.md` + `git log -1` — max 1–2 lines per project. No code is read; the handover files carry the status (see [04 — Conventions](04-conventions.md))
5. **One focus proposal** (at most one secondary), justified from a priorities file the assistant may read but never edit
6. Route into the chosen project

The evening skill is the mirror image (2–3 minutes): walk today's checkboxes, capture carry-overs for tomorrow, close with one calm sentence. Both skills end with a local git commit of the day file — **never a push**; the day files live in a local-only repo.

A sanitized, reusable version is in [`examples/skills/daily-brief/SKILL.md`](../examples/skills/daily-brief/SKILL.md).

## Design decisions worth stealing

**Privacy by design.** The journal is *never* stored by the tool — it lives in a separate notes app. The assistant never displays previous days' journal entries; today's reflection must be fresh. The tool only stores task lists and focus choices. Deciding *what the assistant must not persist* turned out to be as important as deciding what it does.

**Ownership boundaries in file form.** The priorities file is owned by the human: the assistant reads it for the focus proposal but edits it only on explicit request. The boundary is written in `CLAUDE.md` as a hard rule, so every session inherits it.

**Tone as a contract.** The workspace rules specify the assistant's tone: short, precise, subtly motivating — *no nagging, no guilt-tripping over unfinished items*. Unchecked boxes carry over silently. This is written down because the default failure mode of a task assistant is becoming a passive-aggressive to-do list.

**Skippable everything.** Any step can be skipped with a word. The skill is a default path, not a form to fill in.

## Why skills instead of a scheduled job

The brief runs when the human shows up and says good morning — not at 07:00 into the void. That keeps the interaction conversational (you can push back on the focus proposal, ask for details on one project, skip the rest) and means zero infrastructure: the entire assistant layer is markdown files in a git repo plus Claude Code's built-in skill mechanism.
