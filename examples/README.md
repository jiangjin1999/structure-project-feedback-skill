# Worked Examples — A 3-iteration story

[English](README.md) | [简体中文](zh-CN/README.md)

These examples form **one continuous story** of maintaining a real,
well-known open-source project across three rounds of feedback. They are
designed to show the part of the skill that is hardest to demonstrate
with a single isolated example: **how `task_plan.md`, `findings.md`,
`progress.md`, and `changelog.md` accumulate as a long-lived, file-based
memory** for the project across iterations.

## The host project

We use [**openai/codex**](https://github.com/openai/codex) as the host
project — the open-source coding agent CLI from OpenAI (~80k stars,
Apache-2.0). It is a particularly good fit because:

- Codex itself follows the SKILL.md / AGENTS.md spec, so we are using
  the skill on a project that natively understands it.
- It has a real community, real issues, and the kind of feature
  decisions that produce the messy multi-stakeholder feedback this
  skill is meant to absorb.
- Apache-2.0, public — anyone can fork and reproduce the loop.

> Note on PR / issue numbers: we use illustrative IDs (`#PR-4123`,
> `#ISS-4099`, etc.) so the examples are concrete without
> misrepresenting any specific real PR. The feature being designed —
> persistent agent memory across sessions — is a real, recurring topic
> in the codex community.

## The story

We follow a single feature through three iterations:

> **Goal:** add persistent agent memory across `codex` sessions, so that
> AGENTS.md / skill state, findings, and approval decisions can survive
> between runs.

| Iteration | Trigger | Skill role | Key files updated |
|---|---|---|---|
| [**1 — RFC triage**](iteration-1-rfc-triage.md) | A 12-comment GitHub Discussion full of mixed proposals, conflicts, and tangents | Turn the discussion into an actionable RFC plan; record open decisions | `feedback.md`, `next_prompt.md`, `task_plan.md`, `findings.md`, `progress.md`, `changelog.md` |
| [**2 — PR review**](iteration-2-pr-review.md) | The implementation PR gets mixed reviewer comments, and one reviewer **challenges a decision from Iteration 1** | Read the prior `findings.md` before processing; either confirm or open a Q for the challenged decision | Same files updated **additively**, plus `archive/feedback-iter-1.md` |
| [**3 — Post-launch user feedback**](iteration-3-post-launch-feedback.md) | After v0.5 ships, users file a mix of bug reports, feature ideas, and one suggestion that was already deferred in Iteration 1 | Cross-reference the new feedback against existing `findings.md` and the `Backlog` section to avoid re-litigation | Same files; `archive/` now has two snapshots; `changelog.md` shows the full v0.4 → v0.5 → v0.5.1 chain |

## What the story is meant to show

A skill is only useful if it makes the **next** iteration cheaper than
the last. After reading these three examples in order you should be able
to answer:

1. *What did the agent in Iteration 2 know about the choices made in
   Iteration 1, and where did that knowledge live?* → in `findings.md`
   and in the `archive/` snapshot of the prior `feedback.md`.
2. *What stopped Iteration 3 from re-discussing a question already
   resolved earlier?* → the `findings.md` entry from Iteration 1, plus
   the explicit `Backlog` section of the Iteration 1 prompt.
3. *Could a different agent — or a human — pick up the project at
   Iteration 3 with no prior context?* → yes, by reading
   `task_plan.md`, `findings.md`, `progress.md`, `changelog.md`, and
   the latest `next_prompt.md`.

That round-trip is the actual contract this skill enforces.

## Reading order

1. [Iteration 1 — RFC triage from a GitHub Discussion](iteration-1-rfc-triage.md)
2. [Iteration 2 — PR review with a challenge to a prior decision](iteration-2-pr-review.md)
3. [Iteration 3 — Post-launch user feedback with deferred-idea revival](iteration-3-post-launch-feedback.md)

> The feedback contents, file states, and PR/issue identifiers are
> illustrative. Adapt them to your own repository and your team's
> tooling conventions.
