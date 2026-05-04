# Structure Project Feedback

> Turn scattered human feedback after a project iteration into a clear next-iteration prompt, an updated file-based plan, and a reviewable Git diff.

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![SKILL.md](https://img.shields.io/badge/spec-SKILL.md-7c3aed)](structure-project-feedback/SKILL.md)
[![Codex](https://img.shields.io/badge/Codex-CLI-black)](https://github.com/openai/codex)
[![Claude Code](https://img.shields.io/badge/Claude-Code-orange)](https://docs.anthropic.com/claude-code)
[![Cursor](https://img.shields.io/badge/Cursor-supported-22d3ee)](https://cursor.sh)

After every iteration of a writing draft, prototype, slide deck, code change, or research artifact, humans usually return with **messy, half-formed feedback**: corrections, taste changes, vague intent, hard constraints, off-topic backlog ideas. Most AI agents collapse this into a vague "let me try again" — losing source nuance and the audit trail.

`structure-project-feedback` is an AI agent **skill** (`SKILL.md` spec) that gives any compatible agent a clear playbook for that exact moment:

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

---

## How it works

The skill follows a 7-step loop documented in [`structure-project-feedback/SKILL.md`](structure-project-feedback/SKILL.md):

1. **Orient** — read existing instructions and plans.
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
├── LICENSE
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
