---
name: daily-brief
description: Morning routine. Run when the user greets you in the morning or asks to start the day. Carry-overs, cross-project brief, one focus proposal. Hard cap ~5 minutes.
---

# Daily brief

Run the steps in order. **Every step is skippable** — if the user says skip, move on without comment. Tone: short, precise, subtly encouraging. Never nag, never guilt-trip about unfinished items.

## Steps

1. **Open calmly.** One short line to start the day. (Adapt: a reflection prompt, a quote, or nothing — make this step your own.)

2. **Journal nudge.** Remind the user to write their morning note *in their own journal app*. HARD RULE: never store journal content in this system, and never display previous days' notes — today's reflection must be fresh. This tool stores task lists and focus choices only.

3. **Empty the head.** Read yesterday's file in `days/` for unchecked `- [ ]` items; merge with the standing checklist in `checklist.md`. Ask for anything new on the user's mind. Write today's `days/YYYY-MM-DD.md`:

   ```markdown
   # YYYY-MM-DD

   ## Must do
   - [ ] ...

   ## Carry-over
   - [ ] ...

   ## Focus
   <filled in step 5>
   ```

4. **Cross-project brief.** For each active project (listed in the workspace CLAUDE.md): read `docs/progress.md` and `git log -1 --format='%h %s (%cr)'`. Report max 1–2 lines per project. Do NOT read code — the handover file is the status.

5. **One focus proposal.** Propose today's main focus (at most one secondary), justified from the priorities file. HARD RULE: you may read the priorities file, but never edit it unless explicitly asked.

6. **Route.** When the user picks a focus, follow the workspace router: read that project's `CLAUDE.md` + `docs/progress.md` + active `docs/todo.md`, summarize in 2–3 lines, continue under the project's own rules.

## Close

Commit the day file in the local repo (`git add days/ && git commit`). Never push — this repo has no remote by design.
