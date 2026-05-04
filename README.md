# Structure Project Feedback Skill

This repository contains a Codex skill named `structure-project-feedback`.

The skill helps an AI agent turn scattered human feedback after a project iteration into a structured next-iteration prompt, update file-based planning records, preserve raw notes, and review changes with Git.

## Contents

- `structure-project-feedback/SKILL.md`: the skill instructions.
- `structure-project-feedback/references/feedback-prompt-template.md`: reusable next-iteration prompt template.
- `structure-project-feedback/agents/openai.yaml`: UI metadata for skill lists.

## Install

Copy the `structure-project-feedback` folder into your Codex skills directory, usually:

```bash
~/.codex/skills/
```

Then ask Codex to use `$structure-project-feedback` when you want to organize project feedback into a clear revision prompt and update planning files.

No license file is included yet. Add one before broad redistribution if you want to make reuse terms explicit.
