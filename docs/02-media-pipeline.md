# 02 — The media pipeline

The main production system: a TypeScript CLI that takes a video idea through research, scriptwriting, narration, visual sourcing, assembly, QC and review — producing publishable long-form video (calm documentary/sleep content) with a human approving every irreversible step.

```
new → research → script → [HUMAN GATE: approve script]
    → narrate (TTS) + assets → assemble → qc → metadata → review
    → [HUMAN GATE: approve video] → manual upload
```

One command per stage. Every stage is idempotent and individually re-runnable. The human approves the script before any money is spent, approves the finished video before anything leaves the machine, and performs the upload manually.

## State: file-per-video, no database

Each video is a directory: `project.json` (status + gates), `research.md`, `script.md`, `sources.md`, `narration.mp3`, `subtitles.srt`, `assets/` + `rights.json`, `metadata.json`, `qc-report.json`, `review.html`, `video.mp4`.

- `project.json` is the single source of truth, validated with zod. Status is only ever mutated through one core module — no stage writes state on its own.
- The directory name is a **readable state machine**: `<date>-[PUBLISHED-]<CODE>-<title-slug>`. Scanning the catalog folder tells you the production state of everything without opening a single file. The `PUBLISHED-` marker is inserted on upload, so the folder listing doubles as the production queue.

## Gates are enforced in code, not in prompts

The two approval gates (`approval:script`, `approval:video`) are checked by a central `blockers()` function in the core. Any stage that costs money (TTS) or downloads assets refuses to run while a blocker is present. This is the load-bearing design decision: **an LLM can be talked out of a prompt-level rule; it cannot talk its way past a function that won't execute.**

A sibling pipeline (short-form video) pushes the gate pattern further with two ideas worth stealing:

- **The two-run gate.** The candidate-generation stage runs once, produces options with the LLM, and *halts*. The second run builds the production plan purely from the human's yes/no marks — no LLM involved, and unmarked means no. The LLM proposes; only explicit human marks decide.
- **Hash-voided approval.** The approval records a content hash of the plan. Edit the plan after approving and the gate silently un-approves — you can't sneak changes past a stale approval.

## The rights invariant

Every single asset must have an entry in `rights.json` — source, license, retrieval date — or the pipeline halts. Consequences of making this an invariant rather than a guideline:

- Credit lines in published metadata are **generated from rights data**, never hardcoded — so attribution can't drift out of sync with what was actually used.
- CC-licensed sources carry license info per file (artist + license URL where required).
- AI-generated imagery (Gemini stills, fal.ai video) gets its own provenance line and an honest imagery statement in the metadata. The rule as written in the repo: AI-generated art must never sail under "authentic archival material."

## Automated QC before human review

Before a human ever watches the video, a QC stage checks duration, audio clipping and silences, missing files, missing rights entries, and topic similarity against the back catalog (to prevent accidental self-repetition). Failures halt with an explicit report.

Then a single self-contained `review.html` per video presents everything the reviewer needs — script, audio player, QC report, metadata, rights list — as one approval surface. The gate decision becomes a two-minute read instead of a scavenger hunt.

## Multi-channel via atomic config overlay

Multiple channels run on the same pipeline through JSON config overlays on a default config, with one hard-learned rule: the `channel` and `categories` sections are replaced **atomically, never deep-merged**. Deep-merging channel configs leaks one channel's voice and category settings into another. Each video project is stamped with its channel, and the CLI asserts the stamp matches before any stage runs.

## Provider abstraction — and no LLM SDK

All external services sit behind provider interfaces in the core:

- **LLM:** a subprocess call to `claude -p` (headless Claude Code). There is *no LLM SDK in the dependency tree* — the application code is decoupled from any specific provider, the model is swappable behind one interface, and the pipeline never touches an LLM API key.
- **TTS:** ElevenLabs with word-level timestamps — one call yields both narration audio and an SRT subtitle track.
- **Visuals:** public-domain-first (NASA, Wikimedia) with per-source license handling; AI generation (Gemini stills, fal.ai video) as an explicitly-labeled source.
- **Rendering:** Remotion (React-based video), driven by a shot plan produced from script structure.

The core/interface split is a standing rule: pipeline logic lives in `core/` and must contain nothing CLI-specific, so a web UI can be layered on later without touching the pipeline.

## Ethical lines, written into the repo

Some constraints are documented as permanent decisions, not preferences: no voice cloning or reuse of real people's audio; research-driven original scripts with real, cited sources; human approval on every script and video — deliberately engineered so the output holds up against platform policies on mass-produced content. The docs also encode a platform constraint (a hard duration cap for short-form) in four separate places in code, with a comment forbidding weakening it without an explicit human decision.
