# 05 — The wider workspace

The long-form media pipeline gets its own doc because it's the deepest system, but it's one of several running on the same conventions. This is the tour of the rest — each built with the same spine: file-based state, LLM behind a provider interface, and a human gate before anything irreversible.

## Short-form content machine (transcription → captions → gated clips)

A separate pipeline turns long-form personal recordings into platform-ready short clips:

- **Automatic transcription** with Whisper, plus **synced lyrics** fetched from a public lyrics-timing API for music segments — so both speech and song can be captioned accurately.
- **LLM segmentation**: the transcript is analyzed into candidate clips (hooks, self-contained moments).
- **The two-run gate**: run one generates candidates with the LLM and *halts*. Run two builds the production plan purely from the human's yes/no marks — no LLM in the loop, unmarked = no.
- **Two-stage rendering** via Remotion, burning styled captions into the video.
- A hard platform constraint (a strict duration cap for short-form) is enforced in **four separate places in code**, with a comment forbidding weakening it without an explicit human decision.
- **Hash-voided approval**: the gate records a content hash of the plan; editing the plan after approval silently invalidates the gate.

The long-form pipeline has a shorts machine of its own: a `ministory` stage that extracts moments from finished documentaries and **rewrites them to stand alone** (never bare cut-outs), with the same analyze → approve → produce gate.

## Personal assistant (calendar · email · facts)

A CLI assistant for calendar, inbox and personal facts — currently in the specification/scaffold stage, included here because the architecture is the interesting part:

- **Guarantee by absence**: the OAuth scope technically permits sending email, so the safety guarantee is that *no send path exists in the codebase*. Every outbound action materializes as a proposal file; a separate explicit `confirm` command is the only way anything happens.
- **Tiered model routing**: a cheap model by default, automatic retry on a mid-tier model when structured output fails validation, escalation to the top model only behind an explicit `--deep` flag. Cost control as architecture, not discipline.

## Income & outreach CLI

A pipeline for tracking income opportunities (gigs, clients, applications) end to end:

- **File-per-opportunity state machine** — each opportunity is a directory with a zod-validated manifest, the original posting, notes, and dated LLM-generated drafts.
- **Deterministic scoring in code**: the LLM supplies sub-scores and rationales; the ranking math is code, so scores are reproducible and auditable.
- **Bilingual drafting** from `{{placeholder}}` templates (outreach and cover letters in two languages).
- Nothing is ever sent by the tool: the `sent` status requires `--confirm --via "..."` — an attestation that a human sent it manually, and through what channel.

## Marketing engine

A skills-based marketing layer (copywriting, offer design, pricing, positioning, prospecting, SEO auditing) with per-business workspaces that point at canonical sources instead of duplicating them. The skills themselves are third-party imports — which is exactly why the interesting part is the **governance**:

- Every imported skill was audited **file by file** before activation, logged in the decision log.
- Standing rule: no new skill enters the workspace without the same audit, and shell recipes inside imported skills never run without per-instance approval.
- A dated `learnings.md` accumulates generalizable corrections, so feedback given once persists across sessions.

## Channel-scale content research

A research CLI that ingests an **entire public video channel** and produces one ranked report: every video is transcribed (platform captions when available, otherwise local `whisper.cpp` — free and offline), each transcript gets a zod-validated LLM summary, and a single synthesis call produces an overall assessment with categories and top picks, rendered to a static HTML report with no LLM in the final step.

The cost discipline is the interesting part:

- **Free before paid**: transcription is local; the LLM enters only after all transcripts exist, as a separate command — so a full channel scan costs nothing until you decide otherwise.
- **Consent before spend**: the summarize step prints *how many LLM calls it is about to make* before running. The same stop-before-it-costs DNA as the media pipeline's gates, applied to API spend.
- **Idempotent by default**: already-processed videos are skipped, so `--limit 10` today and `--all` tomorrow only pays for the delta.
- **Token hygiene in config**: transcripts are truncated to a configured character budget before any LLM call — input validation as a cost control, not an afterthought.

Built for evaluating content claims at scale (are the methods this channel teaches actually worth testing?) — a reminder that agentic tooling is as useful for *analyzing* content as for producing it.

## Job-application tooling

A deliberately thin bash + Python toolchain: render a tailored application PDF from a job posting, build the full submission package, and keep a **dedupe ledger** (`check` before writing, `add` after the human has sent it). Submission is human-only — browser-automating application forms was evaluated and rejected as net-negative. The lesson worth keeping: sometimes the right amount of automation is *less*.

## Why show the breadth

Any single pipeline could be a weekend demo. The point of this workspace is that the same small set of patterns — handover files, gates in code, provider interfaces, file-based state, the maturity ladder — has now carried **seven different systems** across unrelated domains (long-form video, short-form video, personal admin, outreach, marketing, channel research, job applications) without the architecture changing. That's the actual evidence that the patterns work.
