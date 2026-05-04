# Example 1 — Slide deck feedback

[English](slide-deck-feedback.md) | [简体中文](zh-CN/slide-deck-feedback.md)

> Scenario: a designer has just shared a 12-slide draft deck with their
> teammate. The teammate replies with one paragraph of mixed feedback
> (corrections, taste, off-topic ideas, an open question, and a "don't change this").
> The agent has the `structure-project-feedback` skill installed.

---

## 1. Raw human feedback (verbatim)

> "First two slides feel right. Slide 3 is too dense — too many bullets.
> Can you also try a different color theme? Maybe something less corporate.
> What's our message for slide 4 again? I forgot what we agreed on.
> Please don't change the typography we picked.
> The case-studies idea is interesting but let's defer it for now.
> Also one of the icons on slide 7 is broken (the one in the top-right).
> Oh and we promised this to leadership by Friday."

## 2. What the skill does, step by step

### Step 1 — Orient

```
$ git status --short
 M slides/main.key
?? notes.txt
```

The agent reads existing project files: `task_plan.md`, `progress.md`,
`changelog.md`, then peeks at the deck itself.

### Step 2 — Preserve

The verbatim paragraph is appended to `feedback.md` with a date heading
**before** any summarization:

```markdown
## 2026-04-12 — review of v3 draft deck

> First two slides feel right. Slide 3 is too dense — too many bullets.
> ... (verbatim) ...
```

The previous `feedback.md` (if it existed and was reduced) is snapshotted
to `archive/feedback-2026-04-09.md` first.

### Step 3 — Classify

| Bucket | Content |
|---|---|
| **Must change now** | Reduce slide 3 density. Try alternative color themes. Fix broken icon on slide 7 (top-right). |
| **Keep stable** | Slides 1–2 (approved). Existing typography. |
| **Intent** | Less corporate vibe overall. |
| **Constraint** | Hard deadline: Friday (leadership commitment). |
| **Question** | Slide 4 message — what did we agree on? Needs clarification before redesign. |
| **Backlog** | Case-studies section. |

## 3. Generated `next_prompt.md`

```markdown
# Next Iteration Prompt — Deck v4

## Context
Draft deck v3 (12 slides) reviewed on 2026-04-12. Goal: present to leadership Friday.

## Must Change Now
- Slide 3: reduce density to ≤ 4 bullets, ≤ 25 words each.
- Color theme: propose 2–3 alternatives (less corporate; warmer / softer palette welcome) before applying any.
- Slide 7: replace the broken top-right icon. Verify all other icons render.

## Keep Stable
- Slides 1–2 (approved as-is).
- Typography choices from v3 — do not change fonts, weights, or hierarchy.

## Intent
- Overall vibe should feel less corporate — leaning approachable, modern.

## Constraints
- Hard deadline: Friday EOD (leadership review).
- Color theme alternatives must keep WCAG AA contrast for body text.

## Open Questions
- Slide 4 core message — please confirm before redesign. Two candidates from notes.txt:
  (a) "we ship faster than we plan" or (b) "we plan in writing, ship in steps".

## Backlog (do not implement this round)
- Case-studies section.

## Acceptance Checks
- Slide 3 word count ≤ 100.
- Color variants saved as `theme-A.key` and `theme-B.key`.
- All icons load (visual sweep + export-to-PDF check).
- Friday delivery file: `deck-v4-final.key`.
```

## 4. Updated planning files (additive only)

### `task_plan.md` (excerpt — diff view)

```diff
 ## In Progress
-- [ ] Build draft deck v3
+- [x] Build draft deck v3 (reviewed 2026-04-12)
+- [ ] Reduce density of slide 3
+- [ ] Produce 2–3 color theme variants
+- [ ] Replace broken icon on slide 7

 ## Open Questions
+- [ ] Confirm slide 4 core message before redesign

 ## Backlog
+- [ ] Case-studies section (deferred from v4)
```

### `progress.md` (excerpt)

```markdown
## 2026-04-12

- v3 draft delivered for review; 6 actionable items returned.
- Status: slides 1–2 approved; slide 3 + slide 7 need fixes; color theme exploration required; slide 4 blocked on clarification.
- Next milestone: deck-v4 by Friday EOD (hard deadline).
```

### `changelog.md`

```markdown
## 2026-04-12

- Reviewed deck v3.
- Updated `task_plan.md`: 4 new tasks; case-studies moved to backlog.
- Generated `next_prompt.md` for the v4 iteration.
- No artifact changes yet.
```

## 5. Git review

```
$ git status --short
 M task_plan.md
 M progress.md
 M changelog.md
?? feedback.md
?? next_prompt.md

$ git diff --stat
 changelog.md   |  6 ++++++
 progress.md    |  5 +++++
 task_plan.md   |  9 +++++++--
 3 files changed, 18 insertions(+), 2 deletions(-)
```

The agent reports the changed files and the unresolved question, then waits
for the user to (a) answer the slide-4 question, or (b) ask it to start
implementing the must-change items, or (c) just commit the planning updates.

---

> **Takeaway** — the original 7-thought paragraph from the user is now:
> a verbatim record, a classified table, an actionable prompt with acceptance
> checks, and an additive update to the project's plan files — all reviewable
> in a single Git diff.
