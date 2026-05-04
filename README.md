# Structure Project Feedback

**English** | [简体中文](README.zh-CN.md)

<p align="center">
  <img src="docs/cover.png" alt="Structure Project Feedback — turn scattered feedback into a clear next-iteration prompt" width="100%" />
</p>

> Turn scattered human feedback after a project iteration into a clear next-iteration prompt, an updated file-based plan, and a reviewable Git diff.

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![SKILL.md](https://img.shields.io/badge/spec-SKILL.md-7c3aed)](structure-project-feedback/SKILL.md)
[![Codex](https://img.shields.io/badge/Codex-CLI-black)](https://github.com/openai/codex)
[![Claude Code](https://img.shields.io/badge/Claude-Code-orange)](https://docs.anthropic.com/claude-code)
[![Cursor](https://img.shields.io/badge/Cursor-supported-22d3ee)](https://cursor.sh)

After every iteration of a writing draft, prototype, slide deck, code change, or research artifact, humans usually return with **messy, half-formed feedback**: corrections, taste changes, vague intent, hard constraints, off-topic backlog ideas. Most AI agents collapse this into a vague "let me try again" — losing source nuance and the audit trail.

`structure-project-feedback` is an AI agent **skill** (`SKILL.md` spec) that gives any compatible agent a clear playbook for that exact moment:

0. **Bootstrap** *(first use only)* — read the project's existing structure and inherit its naming, instead of forcing a fresh file kit on top of your repo.
1. **Preserve** the user's raw words before summarizing.
2. **Classify** what must change vs. what should stay stable.
3. **Generate** a next-iteration prompt that is auditable and portable across agents.
4. **Update** file-based planning records (`task_plan.md`, `findings.md`, `progress.md`, `changelog.md`).
5. **Review** with Git so the human can see exactly what changed.

It is compatible with any AI coding agent that follows the SKILL.md / AGENTS.md convention: **Codex CLI, Claude Code, Cursor, Gemini CLI**, and similar.

---

## Why use this skill

| Pain point | Without the skill | With the skill |
|---|---|---|
| Half-formed feedback ("the tone feels off + can you also try X?") | Agent rewrites everything; original wording is lost | Raw feedback archived; structured by category; only what was requested actually changes |
| Multiple feedback rounds | Plan drifts, old decisions are forgotten | `findings.md` + `changelog.md` keep decisions stable across rounds |
| Switching between AI tools | Each agent re-discovers context from scratch | The generated next-iteration prompt is tool-agnostic and reusable |
| "What changed?" after an iteration | Unstructured diff, hard to review | Git diff + the file kit make every change explicit |
| Introducing the skill into a repo that already has its own `TODO.md`, `ROADMAP.md`, `DECISIONS.md`, etc. | The agent creates duplicate plan files; two sources of truth drift apart | **Bootstrap step** maps existing files to the skill's roles and reuses them in place |

---

## Quick install

### Codex CLI

```bash
mkdir -p ~/.codex/skills
cd ~/.codex/skills
git clone https://github.com/jiangjin1999/structure-project-feedback-skill.git _spf
mv _spf/structure-project-feedback ./
rm -rf _spf
```

Then trigger from a chat:

```text
Use $structure-project-feedback to organize my scattered project feedback
into a clear next-iteration prompt and update the planning files.
```

### Claude Code

```bash
mkdir -p ~/.claude/skills
cd ~/.claude/skills
git clone https://github.com/jiangjin1999/structure-project-feedback-skill.git _spf
mv _spf/structure-project-feedback ./
rm -rf _spf
```

For project-scoped install, use `.claude/skills/` inside the project instead of `~/.claude/skills/`.

### Cursor

System-wide:

```bash
mkdir -p ~/.cursor/skills
cd ~/.cursor/skills
git clone https://github.com/jiangjin1999/structure-project-feedback-skill.git _spf
mv _spf/structure-project-feedback ./
rm -rf _spf
```

Project-scoped: drop the `structure-project-feedback/` folder under `.cursor/skills/` of your repo.

### Gemini CLI / OpenClaw / generic SKILL.md tools

Place the `structure-project-feedback/` directory wherever your agent reads skill manifests. The skill is a pure prompt + reference bundle — no runtime, no MCP server, no external API.

### Add a trigger rule to `AGENTS.md`

```md
## Skills

- structure-project-feedback: Use when the user reacts to a draft, prototype,
  code result, doc, deck, or plan with multiple unstructured comments.
  Produces a next-iteration prompt and updates planning files
  (`task_plan.md`, `findings.md`, `progress.md`, `changelog.md`).
```

---

## When to use it

- The user reviews a result and gives **mixed feedback** (corrections + new ideas + style notes + questions).
- A **multi-round project loop** where the next-iteration prompt should be reusable across AI tools.
- You maintain `task_plan.md`, `findings.md`, `progress.md`-style records and want them updated consistently.
- You need an **audit trail** (Git diff) of what each feedback round changed.

## When NOT to use it

- A single, atomic, unambiguous instruction (e.g. "rename `foo` to `bar`") — just edit directly.
- A brand-new project with no prior artifact yet — use a planning / brainstorming skill first.
- Pure code refactors with no human-language feedback — stick to your normal code-edit loop.

---

## Bootstrap from an existing project

The first time the skill runs in a project that already has content — code, docs, notes, a `TODO.md`, an `AGENTS.md`, a `ROADMAP.md`, a `docs/decisions/` folder, or anything else — it **does not** overwrite your world with a generic file kit. It adapts to what is already there:

1. **Scans** the top-level tree, reads `README`, `AGENTS.md`, `CONTRIBUTING.md`, and the 10 most recent commits (read-only).
2. **Maps** existing files to the skill's roles. For example:

   | Skill role | Accepted existing files |
   |---|---|
   | `feedback.md` | `notes.md`, `REVIEW.md`, `comments.md`, `issues/*.md`, … |
   | `task_plan.md` | `TODO.md`, `PLAN.md`, `ROADMAP.md`, `backlog.md`, `tasks.md`, … |
   | `findings.md` | `DECISIONS.md`, `docs/decisions/`, `ADR/`, `rationale.md`, … |
   | `progress.md` | `PROGRESS.md`, `status.md`, `weekly.md`, `journal.md`, … |
   | `changelog.md` | `CHANGELOG.md`, `HISTORY.md`, `RELEASE_NOTES.md`, … |
   | `next_prompt.md` | `prompt.md`, `next.md`, `instructions.md`, … |

3. **Infers the project type** (library / paper / deck / web app / research notebook / docs-only, etc.) from evidence (`pyproject.toml`, `*.tex`, `*.key`, `package.json`, `notebooks/`, ...).
4. **Seeds only what is missing**, using the naming style the repo already uses (`UPPERCASE.md` vs. `lowercase.md` vs. `kebab-case.md`).
5. **Records the mapping** under `## Project shape — auto-detected (<date>)` inside `findings.md` (or the mapped equivalent). That block is the idempotency marker — bootstrap runs **at most once** per project.

If anything is ambiguous (e.g. two plausible `task_plan` candidates), the skill asks **one** consolidated question with both surfaced — never a storm of small questions. You can explicitly re-run with *"re-bootstrap the project shape"* after a major repo restructure.

---

## What the skill produces

Given a feedback round, the skill produces a consistent set of artifacts:

```text
your-project/
├── feedback.md            ← raw feedback, preserved verbatim
├── archive/
│   └── feedback-2025-04-12.md   ← historical raw notes
├── task_plan.md           ← work items with statuses
├── findings.md            ← decisions, constraints, lessons
├── progress.md            ← what changed this round, what remains
├── next_prompt.md         ← reusable prompt for the next AI pass
└── changelog.md           ← human-readable history
```

Plus a Git diff you can review before committing.

---

## A worked example

**Input — a user pastes 6 bullet points after seeing a draft slide deck:**

> "The first 2 slides feel right; slide 3 is too dense; can you also try a different color theme; what's our message for slide 4 again? please don't change the typography we chose; let's defer the case-studies idea for now."

**Output the skill produces:**

`feedback.md` keeps all 6 bullets verbatim.

`next_prompt.md`:

```markdown
# Next Iteration Prompt

## Must Change Now
- Reduce density of slide 3 (target ≤ 4 bullets, ≤ 25 words each).
- Try alternative color themes (propose 2–3 options before applying).

## Keep Stable
- Slides 1–2 (approved).
- Existing typography choices.

## Open Questions
- Slide 4 message — needs clarification before redesign.

## Backlog
- Case-studies section.

## Acceptance Checks
- Slide 3 word count ≤ 100.
- Color theme A/B variants saved as separate files.
```

`task_plan.md`, `progress.md`, and `changelog.md` are updated additively. Anything large that would otherwise be overwritten is snapshotted to `archive/` first.

The same loop works for code review feedback, manuscript comments from a reviewer, design crits on a UI prototype, or product feedback on a prototype demo.

### Full walkthrough — one continuous story across 3 iterations

The examples follow a single feature being designed for the real
[`openai/codex`](https://github.com/openai/codex) project (the open-source
~80k★ coding agent CLI from OpenAI): **persistent agent memory across
sessions**. Each iteration shows how `task_plan.md`, `findings.md`,
`progress.md`, and `changelog.md` accumulate as the project's long-lived
file-based memory.

| # | Iteration | What's special |
|---|---|---|
| 1 | [RFC triage from a 12-comment GitHub Discussion](examples/iteration-1-rfc-triage.md) | Bootstraps the file kit; logs 4 open decisions |
| 2 | [PR review with a challenge to a prior decision](examples/iteration-2-pr-review.md) | Reads `findings.md` first; **re-opens** Q1 instead of silently flipping it |
| 3 | [Post-launch user feedback with deferred-idea revival](examples/iteration-3-post-launch-feedback.md) | Recognizes already-deferred ideas; enforces a constraint set 1 month earlier without anyone re-stating it |

See [examples/README.md](examples/README.md) for the index.

---

## How it works

The skill follows a loop documented in [`structure-project-feedback/SKILL.md`](structure-project-feedback/SKILL.md):

0. **Bootstrap** *(first run per project)* — scan the project, map existing files to skill roles, record the mapping under `## Project shape — auto-detected`.
1. **Orient** — read existing instructions and plans (using the mapping).
2. **Preserve** — archive raw feedback before any summarization.
3. **Classify** — Must change / Keep stable / Intent / Evidence / Constraint / Question / Backlog.
4. **Convert** — produce a next-iteration prompt with acceptance checks.
5. **Update** — sync the plan files.
6. **Execute** *(optional)* — only if the user explicitly asks to implement.
7. **Review** — `git status`, `git diff`, stage, commit if requested.

A reusable prompt skeleton lives at [`structure-project-feedback/references/feedback-prompt-template.md`](structure-project-feedback/references/feedback-prompt-template.md).

---

## Compatibility

| Agent | Supported | Default install path |
|---|---|---|
| Codex CLI | ✅ | `~/.codex/skills/` |
| Claude Code | ✅ | `~/.claude/skills/` (or project `.claude/skills/`) |
| Cursor | ✅ | `~/.cursor/skills/` (or project `.cursor/skills/`) |
| Gemini CLI | ✅ | per SKILL.md convention |
| OpenClaw / Trae / generic SKILL.md tools | ✅ | per skill loader spec |

The skill is plain markdown — no language runtime, no MCP server, no external API.

---

## Repository layout

```text
structure-project-feedback-skill/
├── README.md
├── README.zh-CN.md
├── LICENSE
├── docs/
│   └── cover.png
├── examples/
│   ├── README.md
│   ├── iteration-1-rfc-triage.md
│   ├── iteration-2-pr-review.md
│   ├── iteration-3-post-launch-feedback.md
│   └── zh-CN/
│       ├── README.md
│       ├── iteration-1-rfc-triage.md
│       ├── iteration-2-pr-review.md
│       └── iteration-3-post-launch-feedback.md
└── structure-project-feedback/
    ├── SKILL.md
    ├── agents/
    │   └── openai.yaml
    └── references/
        └── feedback-prompt-template.md
```

---

## FAQ

**Does the skill work if my repo is not a Git repo?**
Yes. The Git review step is a soft guideline. If `git` is unavailable, the skill still produces the structured prompt and the file updates.

**What if my project does not use `task_plan.md` etc.?**
The skill prefers your repo's existing file names. If nothing exists, it creates the smallest useful subset.

**Will it overwrite my existing plan files?**
No. It updates them additively and snapshots large reductions to `archive/` first.

**Does it work in non-English projects?**
Yes. The skill instructs the agent to keep the user's language unless the user asks otherwise.

**How is this different from a generic "planning" skill?**
A planning skill helps you decompose work *before* the first iteration. This skill is specifically for the moment *after* an artifact exists and a human reacts to it with messy feedback. The two compose well.

**I already have my own `TODO.md` / `DECISIONS.md` / `CHANGELOG.md`. Will this skill duplicate them?**
No. The **Bootstrap** step (Step 0) detects existing files and reuses them in place — it only creates new files for roles that are genuinely missing, and uses whatever naming style (`UPPERCASE.md`, `lowercase.md`, `kebab-case.md`) your repo already prefers.

**What if the bootstrap maps a file wrong?**
The mapping is written to `findings.md` (or its mapped equivalent) under `## Project shape — auto-detected`. Edit it once, tell the agent "re-bootstrap the project shape", and it will re-read that block on the next run.

---

## Contributing

Issues, suggestions, and PRs are welcome — especially:

- New trigger phrases for `AGENTS.md`.
- Worked examples in additional languages.
- Refinements to the next-iteration prompt template.

Please keep examples generic so the skill stays reusable across domains.

---

## License

[MIT](LICENSE) — free to use, modify, and redistribute.

---

If this skill saved you time, please star the repo and share with anyone running multi-round AI iteration loops.
