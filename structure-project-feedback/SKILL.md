---
name: structure-project-feedback
description: Structure scattered human feedback after a project iteration into a clear next-iteration prompt, update file-based plans and progress records, preserve raw notes with archives, review changes with Git, and optionally commit or prepare publication. On first use in an existing project, bootstrap by reading the current project structure (code, docs, non-standard plan files) and inherit the repository's conventions instead of forcing a fresh file kit. Use when a user reacts to a draft, prototype, analysis, code result, document, deck, website, or plan with fragmented comments and wants the AI to organize intent, revise the plan, maintain task_plan.md/findings.md/progress.md/changelog.md-style files, or create a reusable prompt for the next round.
version: 1.1.0
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

# Structure Project Feedback

## Overview

Use this skill to turn post-result feedback into an auditable project loop: preserve what the human actually said, infer the next revision cleanly, update the plan files, and use Git to make the state reviewable.

The skill is agent-agnostic (Codex CLI, Claude Code, Cursor, Gemini CLI, and any other SKILL.md-compatible tool).

## When to use this skill

Activate the skill when **all** of these hold:

- An artifact already exists (a draft, deck, prototype, code change, manuscript, plan, design).
- The user has just reacted to it with **multiple, partly unstructured comments** — corrections mixed with taste, intent, questions, or off-topic ideas.
- The user wants the next iteration to be deliberate (not a free-form retry) and ideally reusable across AI tools.

Typical trigger phrases include: "organize my feedback", "make a prompt for the next round", "update the plan with this feedback", "what should change next?", or simply pasting a block of mixed comments after seeing a result.

## When NOT to use this skill

- A single atomic instruction with no ambiguity (e.g. "rename `foo` to `bar`"). Edit directly.
- The project has no prior artifact yet. Use a planning or brainstorming skill first.
- Pure code refactor with no human-language feedback. Stick to the normal code-edit loop.
- The user explicitly asks for an immediate edit and does not want a structured plan.

## Bootstrap from an existing project (Step 0 — only on first use per project)

When the skill is activated on a project that **already has content** (code, docs, notes, early plans under non-standard names, a README, an AGENTS.md, etc.) and the standard file kit (`task_plan.md`, `findings.md`, `progress.md`, `changelog.md`) is **not yet** present, do **not** start by creating a fresh, generic file kit. Adapt to what is already there.

Trigger condition: any of the following are true when the skill first runs in a project:
- The repo has at least one non-empty file other than `.git/`.
- There is no file named exactly `task_plan.md`, `findings.md`, or `progress.md`.
- There is a `README`, `AGENTS.md`, `CONTRIBUTING.md`, `CLAUDE.md`, `CURSOR.md`, package manifest (`package.json`, `pyproject.toml`, `Cargo.toml`, `go.mod`, `*.tex`, `*.Rmd`, etc.), an `issues/` folder, a `docs/` folder, or any file whose name matches the patterns listed below.

Action sequence:

1. **Scan the project shape** (read-only).
   - Run `git ls-files | head -200` or list the top-level tree; identify directories like `src/`, `docs/`, `paper/`, `slides/`, `notebooks/`, `examples/`, `issues/`, `.github/`.
   - Read up to 3 anchor files if present: the root `README*`, `AGENTS.md`, and any `CONTRIBUTING.md`.
   - Inspect the most recent 10 commits with `git log --oneline -10` (skip silently if not a Git repo) to learn message conventions.

2. **Map existing files to the skill's file kit.**
   Use the following equivalence table. If a hit is found, the skill will **reuse that file** instead of creating a new one. Multiple hits are allowed; pick the most active one (most recent modification wins).

   | Skill role | Existing file patterns to accept |
   |---|---|
   | `feedback.md` | `feedback.md`, `FEEDBACK.md`, `notes.md`, `NOTES.md`, `REVIEW.md`, `review-notes.md`, `comments.md`, `issues/*.md` |
   | `task_plan.md` | `task_plan.md`, `TODO.md`, `TODOS.md`, `PLAN.md`, `plan.md`, `ROADMAP.md`, `tasks.md`, `backlog.md`, `.github/ISSUE_TEMPLATE/*` |
   | `findings.md` | `findings.md`, `DECISIONS.md`, `decisions/`, `docs/decisions/`, `ADR/`, `adr/`, `rationale.md` |
   | `progress.md` | `progress.md`, `PROGRESS.md`, `status.md`, `weekly.md`, `updates.md`, `journal.md` |
   | `next_prompt.md` | `next_prompt.md`, `prompt.md`, `PROMPT.md`, `next.md`, `instructions.md`, `AGENTS.md` (read-only reference, do not overwrite) |
   | `changelog.md` | `CHANGELOG.md`, `changelog.md`, `HISTORY.md`, `RELEASE_NOTES.md` |

3. **Seed only what is missing.**
   - If a role already has a mapped file, keep the existing name. Never rename to the canonical form silently.
   - If a role is truly absent and will be needed for this feedback round, create the smallest reasonable file using the **repo's existing naming style** (e.g. if the repo uses `UPPERCASE.md`, create `FEEDBACK.md` rather than `feedback.md`).
   - Never touch `README*`, `LICENSE`, `AGENTS.md`, or package manifests.

4. **Infer the project type** from evidence and note it in findings. Examples:
   - `pyproject.toml` + `tests/` → Python library / app.
   - `*.tex` + `refs.bib` → academic paper project.
   - `slides/` + `*.key` / `*.pptx` → presentation project.
   - `app/` + `package.json` + `tailwind.config.*` → web app.
   - `notebooks/` + `data/` → research / data-analysis project.
   - Only markdown + no code → documentation / knowledge-base project.

5. **Record the bootstrap result** under a clearly marked section in `findings.md` (or the mapped equivalent):

   ```markdown
   ## Project shape — auto-detected (<YYYY-MM-DD>)
   - Project type: <inferred type>
   - File-kit mapping:
     - feedback    → <existing path or "(new) <path>">
     - task_plan   → <…>
     - findings    → <…>
     - progress    → <…>
     - next_prompt → <…>
     - changelog   → <…>
   - Naming style: <lowercase.md | UPPERCASE.md | kebab-case.md | mixed>
   - Commit message style: <conventional | prose | mixed>
   - Language of prior notes: <en | zh | …>
   - Unsure about: <items that need user confirmation in one pass>
   ```

6. **Ask one consolidated question if needed.** If any single mapping is ambiguous in a way that would cause harm (e.g. two plausible `task_plan.md` candidates), ask the user once with both candidates surfaced — do **not** split into many small questions.

7. **Proceed to the standard feedback loop** (Step 1 onwards), now using the mapped file names throughout.

Bootstrap must be performed **at most once per project**. On subsequent invocations, the skill skips this step (the `## Project shape — auto-detected` block in findings is the idempotency marker). If the user later wants to re-bootstrap (e.g. after a large repo restructure), they can explicitly ask: "re-bootstrap the project shape".

## Core Workflow

1. Orient in the project.
   - If this is the first use of the skill in this project (no `## Project shape — auto-detected` block exists in `findings.md` or its mapped equivalent), run **Bootstrap from an existing project** (Step 0 above) first.
   - Read local instructions first: `AGENTS.md`, `README.md`, project-specific skill files, or equivalent.
   - Inspect the current artifact or result the user is reacting to when it is available.
   - Read existing planning files before editing. Use the file names recorded in the bootstrap mapping (falling back to defaults `task_plan.md`, `findings.md`, `progress.md`, `prompt.md` / `next_prompt.md`, `changelog.md`, `feedback.md`).
   - Run `git status --short` before edits so unrelated work is visible. Skip silently if the project is not a Git repo.

2. Preserve raw feedback.
   - Keep the user's original wording somewhere durable before summarizing or replacing it.
   - If reducing a long feedback note, copy the full old version into `archive/` first (file naming: `feedback-YYYY-MM-DD.md` or `feedback-YYYY-MM-DD-<slug>.md`).
   - If the repo has no feedback file, create or use the smallest reasonable one, such as `feedback.md` or a clearly named project note file.

3. Classify the feedback.
   - Separate requested changes, observed problems, intent or taste, hard constraints, examples or references, open questions, and backlog ideas.
   - Preserve uncertainty explicitly instead of converting every comment into a confident requirement.
   - Treat the latest explicit user instruction as highest priority, followed by latest raw feedback, stable project rules, the current plan, generated prompts, and older changelog entries.

4. Convert feedback into a next-iteration prompt.
   - State the project context and current state briefly.
   - List what must change now, what should stay stable, and what should be deferred.
   - Include acceptance checks so the next AI pass can verify whether the revision addressed the feedback.
   - Use the user's language unless the user asks for another language.
   - For a reusable template, read `references/feedback-prompt-template.md`.

5. Update the file-based plan.
   - Update `task_plan.md` with revised work items and statuses.
   - Update `findings.md` with decisions, constraints, and lessons learned.
   - Update `progress.md` with what changed this round and what remains.
   - Update `changelog.md` with date, summary, affected files, and whether the framework, current prompt, or project content changed.
   - Update `prompt.md` or `next_prompt.md` if the user needs a prompt they can send directly to another AI.
   - Updates are additive. Never silently overwrite a stable file — snapshot it to `archive/` first.

6. Execute only when requested or clearly implied.
   - If the user asks to implement the next revision, carry the plan into the artifact or code.
   - If the user only asks for organization, stop after producing the structured prompt and file updates.
   - Ask a concise question only when an ambiguity would materially change direction or risk losing source material.

7. Review with Git.
   - Run `git status --short` and `git diff` after edits.
   - Stage only the intended files.
   - Commit meaningful completed changes when the user asked for persistence or the repository's instructions require it.
   - Use concise messages such as `update: structure project feedback loop` or `docs: refine next iteration prompt`.
   - Push or open a PR only when the user asks for publication or the repo workflow requires it.

## File Kit

Always prefer the **bootstrap mapping** recorded under `## Project shape — auto-detected` (see Step 0). If no mapping exists yet, prefer whatever file names the repository already uses. Only when no equivalent exists should the skill fall back to the canonical names below. Use this canonical set as the minimum useful subset when starting from scratch:

- `task_plan.md`: durable plan and current task breakdown.
- `findings.md`: decisions, constraints, evidence, and lessons.
- `progress.md`: completed work, current status, blockers, and next steps.
- `feedback.md`: raw or lightly structured human feedback inbox.
- `prompt.md` or `next_prompt.md`: directly reusable prompt for the next AI pass.
- `changelog.md`: human-readable history.
- `archive/`: full snapshots before large reductions, rewrites, or cleanup.

## Feedback Taxonomy

Use these buckets when reorganizing scattered comments:

- `Must change now`: direct corrections or required revisions.
- `Keep stable`: parts the user likes or does not want disrupted.
- `Intent`: the underlying purpose behind a comment.
- `Evidence`: concrete examples, screenshots, quotes, logs, or files.
- `Constraint`: style, format, tooling, deadline, scope, or compatibility limits.
- `Question`: ambiguity that should be resolved before risky work.
- `Backlog`: useful ideas that should not affect the current pass.

## Output Standards

- Keep raw feedback recoverable. If anything is reduced or rewritten, the original lives in `archive/`.
- Keep the structured prompt action-oriented, not just a summary.
- Make assumptions explicit and easy to revise.
- Avoid overwriting stable plans with transient reactions unless the user clearly changed direction.
- When summarizing to the user, report changed files, validation or Git checks, commit hash if created, and any unresolved question.

## Example interactions

A user paste like this should activate the skill:

> "First two slides feel right, slide 3 is too dense, can you also try a different color theme, what's our message for slide 4 again, please don't change the typography, defer the case-studies idea."

The expected response is a `feedback.md` archive of the verbatim comments, a `next_prompt.md` with `Must change now` / `Keep stable` / `Open questions` / `Backlog` / `Acceptance checks` sections, and updates to any existing `task_plan.md` / `progress.md` / `changelog.md` files.

A single instruction like "fix the typo in line 12" should **not** activate the skill — edit directly instead.

## Resources

- `references/feedback-prompt-template.md`: Load when drafting a reusable next-iteration prompt or when the user's feedback is too scattered to act on directly.
