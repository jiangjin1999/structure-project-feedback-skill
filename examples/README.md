# Worked Examples

[English](README.md) | [简体中文](zh-CN/README.md)

Real, end-to-end walkthroughs of `structure-project-feedback` in action.
Each example shows:

1. The original artifact a human just reviewed.
2. The **raw feedback** the human dropped on the agent (verbatim).
3. How the skill **classified** it.
4. Every file the skill **produced or updated** (`next_prompt.md`,
   `task_plan.md`, `progress.md`, `changelog.md`, `archive/…`).
5. The Git diff a reviewer would see.

| # | Scenario | When to read this |
|---|---|---|
| 1 | [Slide deck feedback](slide-deck-feedback.md) | Mixed style + content + backlog comments after a draft deck. |
| 2 | [Code review feedback](code-review-feedback.md) | A PR with multiple reviewer threads — corrections + nits + larger redesign hints. |
| 3 | [Manuscript revision feedback](manuscript-revision.md) | Academic / long-form writing review with conflicting comments and open questions. |

> All examples are intentionally generic. They are **not** taken from any specific
> private project — feel free to copy and adapt them in your own AGENTS.md.
