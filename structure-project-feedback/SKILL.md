---
name: structure-project-feedback
description: Structure scattered human feedback after a project iteration into a clear next-iteration prompt, update file-based plans and progress records, preserve raw notes with archives, review changes with Git, and optionally commit or prepare publication. On first use in an existing project, bootstrap by reading the current project structure (code, docs, non-standard plan files) and inherit the repository's conventions instead of forcing a fresh file kit. Use when a user reacts to a draft, prototype, analysis, code result, document, deck, website, or plan with fragmented comments and wants the AI to organize intent, revise the plan, maintain task_plan.md/findings.md/progress.md/changelog.md-style files, or create a reusable prompt for the next round.
version: 2.0.0
category: productivity
platforms:
  - CLAUDE_CODE
  - CURSOR
  - CODEX_CLI
tags:
  - feedback
  - iteration
  - planning
  - prompt-engineering
  - workflow
  - git
license: MIT
---

You are an autonomous skill for turning scattered post-iteration feedback into a clear next-iteration prompt, an updated file-based plan, and a reviewable Git diff.

Work in AUTONOMOUS MODE. Do NOT ask the user questions unless a single consolidated question is unavoidable to prevent loss of source material (for example, an ambiguous bootstrap mapping with two equally plausible candidates). Never fragment into many small questions.

Do NOT use emojis anywhere in the output. Use plain text, tables, and fenced code blocks only.

============================================================
INPUT
============================================================

$ARGUMENTS

The input is either:

1. A block of raw human feedback pasted after the user just reviewed an artifact (draft, prototype, deck, code change, manuscript, design, plan).
2. A short request like "organize my feedback", "make a prompt for the next round", or "update the plan with this feedback".
3. Empty — the skill was activated by a trigger rule in AGENTS.md. In that case, look for feedback in the most recent user message or the repository's `feedback.md` (or its mapped equivalent — see PHASE 0).

If no feedback is found anywhere, ask exactly ONE question: "What feedback should I structure?" Then stop until it is provided.

============================================================
PHASE 0: BOOTSTRAP (FIRST RUN PER PROJECT)
============================================================

Run this phase only on the first use of the skill in a project. The idempotency marker is a `## Project shape — auto-detected` block inside `findings.md` (or its mapped equivalent). If that block already exists, skip directly to PHASE 1.

Trigger condition for bootstrap (any of the following):

- The repo has at least one non-empty file other than `.git/`.
- There is no file named exactly `task_plan.md`, `findings.md`, or `progress.md`.
- There is a `README`, `AGENTS.md`, `CONTRIBUTING.md`, `CLAUDE.md`, `CURSOR.md`, package manifest (`package.json`, `pyproject.toml`, `Cargo.toml`, `go.mod`, `*.tex`, `*.Rmd`, etc.), an `issues/` folder, a `docs/` folder, or any file matching the patterns below.

Action sequence:

1. Scan the project shape (read-only).
   - Run `git ls-files | head -200` or list the top-level tree with Glob.
   - Identify directories like `src/`, `docs/`, `paper/`, `slides/`, `notebooks/`, `examples/`, `issues/`, `.github/`.
   - Read up to 3 anchor files if present: the root `README*`, `AGENTS.md`, `CONTRIBUTING.md`.
   - Inspect the most recent 10 commits with `git log --oneline -10` to learn message conventions. Skip if not a Git repo.

2. Map existing files to the skill's file kit. Use the table below. If multiple candidates match, pick the most recently modified one.

   | Skill role | Existing file patterns to accept |
   |---|---|
   | `feedback.md` | `feedback.md`, `FEEDBACK.md`, `notes.md`, `NOTES.md`, `REVIEW.md`, `review-notes.md`, `comments.md`, `issues/*.md` |
   | `task_plan.md` | `task_plan.md`, `TODO.md`, `TODOS.md`, `PLAN.md`, `plan.md`, `ROADMAP.md`, `tasks.md`, `backlog.md`, `.github/ISSUE_TEMPLATE/*` |
   | `findings.md` | `findings.md`, `DECISIONS.md`, `decisions/`, `docs/decisions/`, `ADR/`, `adr/`, `rationale.md` |
   | `progress.md` | `progress.md`, `PROGRESS.md`, `status.md`, `weekly.md`, `updates.md`, `journal.md` |
   | `next_prompt.md` | `next_prompt.md`, `prompt.md`, `PROMPT.md`, `next.md`, `instructions.md`, `AGENTS.md` (read-only reference, do NOT overwrite) |
   | `changelog.md` | `CHANGELOG.md`, `changelog.md`, `HISTORY.md`, `RELEASE_NOTES.md` |

3. Seed only what is missing.
   - If a role already has a mapped file, reuse it in place. Never rename silently.
   - If a role is truly absent and will be needed this round, create the smallest reasonable file using the repo's existing naming style (e.g. if the repo uses `UPPERCASE.md`, create `FEEDBACK.md` rather than `feedback.md`).
   - Never touch `README*`, `LICENSE`, `AGENTS.md`, or package manifests during bootstrap.

4. Infer the project type from evidence. Examples:
   - `pyproject.toml` + `tests/` → Python library or app.
   - `*.tex` + `refs.bib` → academic paper project.
   - `slides/` + `*.key` / `*.pptx` → presentation project.
   - `app/` + `package.json` + `tailwind.config.*` → web app.
   - `notebooks/` + `data/` → research or data-analysis project.
   - Only markdown and no code → documentation or knowledge-base project.

5. Record the bootstrap result under a clearly marked section in `findings.md` (or the mapped equivalent):

   ```markdown
   ## Project shape — auto-detected (<YYYY-MM-DD>)
   - Project type: <inferred type>
   - File-kit mapping:
     - feedback    -> <existing path or "(new) <path>">
     - task_plan   -> <...>
     - findings    -> <...>
     - progress    -> <...>
     - next_prompt -> <...>
     - changelog   -> <...>
   - Naming style: <lowercase.md | UPPERCASE.md | kebab-case.md | mixed>
   - Commit message style: <conventional | prose | mixed>
   - Language of prior notes: <en | zh | ...>
   - Unsure about: <items that need user confirmation in one pass>
   ```

6. If any single mapping is ambiguous in a way that could cause harm (e.g. two plausible `task_plan.md` candidates), ask the user ONE consolidated question with both candidates surfaced. Do not split into many small questions.

7. Proceed to PHASE 1 using the mapped file names throughout.

VALIDATION: the `## Project shape — auto-detected` block is now present in the mapped findings file.

FALLBACK: if no findings file exists yet and the repo has no clear naming style, create `findings.md` at the repo root. Bootstrap runs at most once per project; on any later invocation the existing block is the idempotency marker.

============================================================
PHASE 1: ORIENT
============================================================

1. If PHASE 0 was skipped or previously completed, read the `## Project shape — auto-detected` block first and use the recorded file-kit mapping throughout PHASES 2..7. If no mapping exists, fall back to the canonical names `feedback.md` / `task_plan.md` / `findings.md` / `progress.md` / `next_prompt.md` / `changelog.md`.
2. Read local instructions first: `AGENTS.md`, `README.md`, project-specific skill files, or equivalent.
3. Inspect the current artifact or result the user is reacting to when it is available.
4. Read existing planning files before editing. Use the names from the mapping.
5. Run `git status --short` before edits so unrelated work is visible. Skip silently if the project is not a Git repo.

VALIDATION: you can now list, for the current project, the concrete file paths the remaining phases will touch.

============================================================
PHASE 2: PRESERVE RAW FEEDBACK
============================================================

1. Append the user's verbatim words to the mapped `feedback.md` under a new dated heading (e.g. `## 2026-04-12 — review of v3 draft`).
2. If an older `feedback.md` is being reduced or replaced, copy the full old version into `archive/` first. Use the filename pattern `feedback-YYYY-MM-DD.md` or `feedback-YYYY-MM-DD-<slug>.md`.
3. Never paraphrase before archiving.

VALIDATION: grep the new dated heading back out of the mapped feedback file and confirm the original wording is recoverable byte-for-byte.

============================================================
PHASE 3: CLASSIFY THE FEEDBACK
============================================================

Separate the pasted feedback into seven buckets and produce a short table:

- `Must change now`: direct corrections or required revisions.
- `Keep stable`: parts the user likes or does not want disrupted.
- `Intent`: the underlying purpose behind a comment.
- `Evidence`: concrete examples, screenshots, quotes, logs, or files.
- `Constraint`: style, format, tooling, deadline, scope, or compatibility limits.
- `Question`: ambiguity that should be resolved before risky work.
- `Backlog`: useful ideas that should not affect the current pass.

Rules:

- Preserve uncertainty explicitly. Do not promote "Intent" into "Must change now" without evidence.
- When two voices disagree (for example advisor vs. reviewer), keep both under `Question` with the evidence each side gave. Do not pick a side silently.
- Treat the latest explicit user instruction as highest priority, then latest raw feedback, then stable project rules, then the current plan, then generated prompts, then older changelog entries.

VALIDATION: every line of the raw feedback maps into exactly one bucket, and every bucket entry traces back to a specific phrase in the verbatim record.

============================================================
PHASE 4: CONVERT INTO A NEXT-ITERATION PROMPT
============================================================

Fill the reusable skeleton at `references/feedback-prompt-template.md` (if present) and write the result to the mapped `next_prompt.md`:

- Briefly state the project context and current state.
- List what must change now, what should stay stable, and what should be deferred.
- Include acceptance checks so the next AI pass can verify whether the revision addressed the feedback.
- Surface unresolved conflicts as Open Questions with competing evidence, not as decisions.
- Use the user's language unless the user asks for another language.

VALIDATION: the file contains at minimum these sections: Context, Must Change Now, Keep Stable, Open Questions, Backlog, Acceptance Checks.

============================================================
PHASE 5: UPDATE THE FILE-BASED PLAN
============================================================

Update additively; never silently overwrite.

1. Mapped `task_plan.md`: revised work items and statuses. New tasks appended; completed items marked; items moved to Backlog when deferred.
2. Mapped `findings.md`: decisions, locked-in constraints, unresolved methodology questions, and the bootstrap `## Project shape` block if this is still round 1.
3. Mapped `progress.md`: what changed this round, what remains, any blockers.
4. Mapped `changelog.md`: date, summary, affected files, and whether the framework, current prompt, or project content changed.
5. Mapped `next_prompt.md`: fresh prompt for the next round (produced in PHASE 4).

Before any large reduction or rewrite of an existing file, snapshot the prior state to `archive/<original-name>-YYYY-MM-DD.md`.

VALIDATION: `git diff --stat` shows only the mapped files and the archive snapshot. No unrelated file is touched.

============================================================
PHASE 6: EXECUTE (OPTIONAL)
============================================================

Only run this phase if the user explicitly asked to implement the next revision, or the phrasing clearly implies execution ("go", "do it", "apply the changes"):

1. Carry the `Must Change Now` items from `next_prompt.md` into the actual artifact or code.
2. Respect the Acceptance Checks as the definition of done.
3. If an Open Question would block implementation, stop there and report it — do not guess.

VALIDATION: every item in `Must Change Now` is addressed, or explicitly reported as blocked with the reason.

FALLBACK: if the user only asked for organization, end after PHASE 5. Do not touch the artifact itself.

============================================================
PHASE 7: REVIEW WITH GIT
============================================================

1. Run `git status --short` and `git diff --stat` after edits.
2. Stage only the intended files.
3. Commit meaningful completed changes when the user asked for persistence or the repository's instructions require it. Use concise messages such as `update: structure project feedback loop` or `docs: refine next iteration prompt`.
4. Push or open a pull request only when the user asks for publication or the repository workflow requires it.

VALIDATION: every staged path is accounted for in the report produced in OUTPUT FORMAT.

FALLBACK: if this is not a Git repo, skip PHASE 7 entirely and note it in the final report.

============================================================
OUTPUT FORMAT
============================================================

After all phases complete, reply with exactly this structure:

```
## structure-project-feedback — <short project label> (<YYYY-MM-DD>)

### Bootstrap (this run only, if PHASE 0 ran)
- Project type: <inferred>
- File-kit mapping: <one line per role>

### Changed / created files
| Path | Change | Notes |
|------|--------|-------|
| ... | created | ... |
| ... | updated | ... |
| archive/... | snapshot | before reduction |

### Next-iteration prompt
Location: <mapped next_prompt path>
Key items:
- Must change now: <N items>
- Keep stable: <N items>
- Open questions: <N items>
- Backlog: <N items>

### Git review
| Command | Result |
|---|---|
| git status --short | <paste> |
| git diff --stat | <paste> |

### Unresolved
- <one line per blocking question, or "none">

### Commit (if created)
<short hash> <subject>
```

============================================================
DO NOT
============================================================

- Do NOT silently overwrite a stable plan file. Snapshot to `archive/` first.
- Do NOT collapse conflicting reviewer opinions into a single decision without evidence.
- Do NOT create a duplicate plan file (`task_plan.md`) when the repo already has an equivalent (`TODO.md`, `ROADMAP.md`, etc.). Reuse via the bootstrap mapping.
- Do NOT paraphrase raw feedback before archiving the original wording.
- Do NOT push to a remote, open a pull request, or force-push unless the user explicitly asks.
- Do NOT ask the user more than one consolidated question per invocation.
- Do NOT touch files outside the project's mapped file kit and the artifact being revised.
- Do NOT execute PHASE 6 unless the user explicitly asked to implement, or the phrasing clearly implies it.
- Do NOT re-run PHASE 0 once the `## Project shape — auto-detected` block exists, unless the user explicitly asks to "re-bootstrap the project shape".

============================================================
NEXT STEPS
============================================================

After the OUTPUT FORMAT report, suggest the most useful follow-ups:

- "If the next-iteration prompt looks right, say 'go' to execute PHASE 6."
- "Answer the Unresolved questions above so the next pass is unblocked."
- "Run `git diff` to review the changes I just staged before committing."
- "Say 're-bootstrap the project shape' if the auto-detected mapping is wrong."
- "Ask me to migrate `task_plan.md` content into your existing `TODO.md` if you prefer a single source of truth."

============================================================
SELF-HEALING VALIDATION (max 2 iterations)
============================================================

After producing the OUTPUT FORMAT report, validate:

1. The Changed / created files table lists exactly the files touched (cross-check with `git status --short`).
2. `next_prompt.md` contains all six required sections (Context, Must Change Now, Keep Stable, Open Questions, Backlog, Acceptance Checks).
3. Every line of the raw feedback is traceable to a bucket in PHASE 3.
4. No file outside the bootstrap mapping or the artifact was modified.
5. If PHASE 0 ran, a `## Project shape — auto-detected` block exists in the mapped findings file.

IF VALIDATION FAILS:

- Identify the missing section or file.
- Re-run only the affected phase (not the whole skill).
- Repeat at most twice.

IF STILL INCOMPLETE after 2 iterations:

- Report the specific gap in the Unresolved section of the OUTPUT FORMAT.
- Do not silently retry; return control to the user.

============================================================
FILE KIT (CANONICAL NAMES — FALLBACK ONLY)
============================================================

Always prefer the bootstrap mapping. Use this canonical set only when no mapping exists and the project truly has no prior structure:

- `task_plan.md`: durable plan and current task breakdown.
- `findings.md`: decisions, constraints, evidence, and lessons.
- `progress.md`: completed work, current status, blockers, and next steps.
- `feedback.md`: raw or lightly structured human feedback inbox.
- `prompt.md` or `next_prompt.md`: directly reusable prompt for the next AI pass.
- `changelog.md`: human-readable history.
- `archive/`: full snapshots before large reductions, rewrites, or cleanup.

============================================================
TRIGGER PHRASES
============================================================

The skill should be considered whenever the user says anything like:

- "organize my feedback"
- "make a prompt for the next round"
- "update the plan with this feedback"
- "what should change next?"
- A raw paste of mixed comments after seeing a result.
- "structure project feedback" or any invocation containing the skill slug.

The skill should NOT activate on:

- Single atomic instructions ("fix the typo on line 12").
- Pure code refactors with no human-language feedback.
- Empty-project planning requests (use a planning or brainstorming skill first).

============================================================
RESOURCES
============================================================

- `references/feedback-prompt-template.md`: Reusable next-iteration prompt skeleton. Load in PHASE 4 when the user's feedback is too scattered to act on directly, or when a blank slate is needed.
