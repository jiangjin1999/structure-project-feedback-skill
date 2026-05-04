# Structured Feedback Prompt Template

Use this template when converting scattered human feedback into a prompt for the next AI iteration. Keep the user's language unless they request a different one.

```markdown
# Next Iteration Prompt

## Context

[Briefly state the project, current artifact/result, and what the previous iteration tried to achieve.]

## What I Observed

- [Concrete issue, gap, or reaction from the user.]
- [Evidence such as file names, screenshots, examples, logs, or exact phrases.]

## Must Change Now

- [Actionable revision 1.]
- [Actionable revision 2.]

## Keep Stable

- [Parts of the result, plan, structure, style, or assumptions that should not be changed.]

## Intent Behind The Feedback

- [What the user is really optimizing for: clarity, usability, rigor, speed, taste, scope control, etc.]

## Constraints

- [Tooling, format, language, audience, style, deadline, repo workflow, Git rules, or compatibility limits.]

## Open Questions

- [Only include questions that materially affect direction or risk.]

## Backlog

- [Useful ideas to preserve but not implement in this pass.]

## Acceptance Checks

- [Checkable condition 1.]
- [Checkable condition 2.]
- [Relevant command, visual review, diff review, or artifact inspection.]

## Files To Update

- `task_plan.md`: [plan change]
- `findings.md`: [decision or lesson]
- `progress.md`: [status update]
- `prompt.md` or `next_prompt.md`: [new reusable prompt]
- `changelog.md`: [human-readable change record]
- `archive/`: [snapshots needed before cleanup]
```

When the user only wants a prompt, fill the template and stop. When they also want execution, update the files and artifacts, then review with Git.
