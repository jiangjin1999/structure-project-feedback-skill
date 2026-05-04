# Example 3 — Manuscript revision feedback

[English](manuscript-revision.md) | [简体中文](zh-CN/manuscript-revision.md)

> Scenario: a researcher gets back a long-form review of a paper draft.
> The review contains corrections, taste preferences, two **conflicting**
> suggestions, an open methodological question, and a backlog idea for
> a follow-up paper. The agent must preserve every comment without
> collapsing the conflict prematurely.

---

## 1. Raw human feedback (verbatim)

> "Section 1 introduction is fine but you bury the contribution; move the
> three contributions list to the first page if possible. Section 2 related
> work is too long, cut by 30%. Section 3 method — I think you should reframe
> it as an extension of Method X rather than a new framework. Reviewer 2
> however explicitly said do NOT frame it as an extension because the loss
> formulation is genuinely different — please reconcile this.
>
> Section 4 experiments — table 3 is unreadable, reformat. Add a confidence
> interval column.
>
> Section 5 discussion — too speculative, tighten or move speculation to a
> 'limitations and future work' subsection.
>
> Question: should we run the ablation on the larger dataset before
> resubmitting? My instinct is yes, but it costs ~2 days of compute.
>
> Citations: please add [Smith 2024] and [Liu 2024] in section 3.
>
> Side idea for a future paper: a multi-modal version of this. Don't
> work on this now — keep it for later."

## 2. What the skill does, step by step

### Step 1 — Orient

```
$ git status --short
 M paper/main.tex
 M paper/sections/method.tex
?? notes/feedback-2026-04-18.md
```

Reads `task_plan.md` (paper milestones) and `findings.md` (prior decisions
about framing).

### Step 2 — Preserve

Verbatim review goes into `feedback.md`. The previous, shorter version of
`feedback.md` is moved to `archive/feedback-2026-04-09.md` first.

### Step 3 — Classify

| Bucket | Content |
|---|---|
| **Must change now** | Move contributions to first page. Cut Section 2 by 30%. Reformat table 3. Add CI column to table 3. Tighten Section 5 (or split off "limitations and future work"). Add citations [Smith 2024] and [Liu 2024] in Section 3. |
| **Keep stable** | (Implicit) Sections 1 intro and 4 experiments narrative — only the table needs reformat, not the experiments themselves. |
| **Intent** | Make contribution visible early; trim non-essential prose; keep claims defensible. |
| **Evidence** | Conflicting reviews on Method framing (advisor vs. Reviewer 2). |
| **Constraint** | Compute budget: ablation on the larger dataset costs ~2 days. |
| **Question** | (a) Method framing — extension of Method X vs. new framework? (b) Run larger-dataset ablation before resubmission? |
| **Backlog** | Multi-modal extension as a follow-up paper. |

> **Important — conflict preserved, not resolved.** The skill does NOT
> silently pick a side on the framing. It surfaces both reviews in
> `next_prompt.md`'s **Open Questions** with the evidence each side gave.

## 3. Generated `next_prompt.md`

```markdown
# Next Iteration Prompt — Paper revision (round 2)

## Context
Paper draft v3 reviewed on 2026-04-18 by advisor + Reviewer 2.
Target: resubmission within ~2 weeks. Compute budget: ~2 free days.

## Must Change Now
1. Move the three-contributions list to page 1 (introduction).
2. Cut Section 2 (Related Work) by ≈30%. Preserve coverage of [Doe 2023] and [Wang 2024]; trim narrative around historical context.
3. Reformat Table 3 in Section 4 for readability:
   - Single-column-per-method layout.
   - Add a 95% confidence interval column.
   - Bold the best result per row.
4. Tighten Section 5 (Discussion): move all speculative claims into a new "Limitations and Future Work" subsection. Keep only directly supported discussion in the main body.
5. Add citations in Section 3:
   - [Smith 2024] near the loss formulation paragraph.
   - [Liu 2024] near the related theory subsection.

## Keep Stable
- Section 1 introduction prose (only the contribution list moves).
- Section 4 experiments narrative and main result claims.
- Notation and symbols already defined.

## Open Questions (resolve before further rewriting)

### Q1 — Method framing (advisor vs. Reviewer 2 conflict)
- **Advisor** suggests: reframe Section 3 as an extension of Method X.
- **Reviewer 2** explicitly says: do NOT frame as an extension because the loss formulation is genuinely different.
- Evidence to consider:
  - Loss has a new term not present in Method X.
  - The optimization procedure is identical.
  - Empirically beats Method X on 4/5 benchmarks.
- Possible resolutions:
  (a) Frame as extension; note the new loss term as the contribution.
  (b) Frame as new framework; explicitly compare to Method X as the closest prior.
  (c) Hybrid: "We build on Method X (Section 3.1), but introduce a new loss (Section 3.2) that materially changes the formulation."
- **Decide before rewriting Section 3.** Suggested: (c).

### Q2 — Larger-dataset ablation
- Compute cost: ≈2 days.
- Risk if skipped: reviewer can challenge generality.
- Risk if run: delays resubmission by 2 days.
- Recommendation if budget allows: run.

## Backlog (do not work on this round)
- Multi-modal extension paper.

## Constraints
- No new experiments beyond the larger-dataset ablation in Q2.
- Preserve LaTeX label and bibkey conventions already in use.

## Acceptance Checks
- Page 1 includes a numbered "Contributions" list (3 items).
- `wc -w paper/sections/related-work.tex` shows ~30% reduction vs prior commit.
- Table 3 renders without horizontal overflow at 1-column width.
- Both [Smith 2024] and [Liu 2024] appear in `paper/refs.bib` and are cited in Section 3.
- Section 5 has a clearly delimited "Limitations and Future Work" subsection.
- `git diff` reviewable in <10 minutes by a co-author.
```

## 4. Updated planning files (additive only)

### `task_plan.md` (diff view)

```diff
 ## Paper revision
-- [ ] Address round-1 reviewer feedback
+- [x] Address round-1 reviewer feedback
+- [ ] Move contributions to page 1
+- [ ] Trim related work by ~30%
+- [ ] Reformat table 3 + add CI column
+- [ ] Tighten section 5 / split limitations subsection
+- [ ] Add citations [Smith 2024] and [Liu 2024]
+- [ ] (Decision needed) Method framing — see findings.md Q1
+- [ ] (Decision needed) Run larger-dataset ablation — see findings.md Q2

 ## Backlog
+- [ ] Multi-modal extension paper
```

### `findings.md` (excerpt)

```markdown
## 2026-04-18 — Open methodological questions

### Q1 — Method framing
Conflict between advisor and Reviewer 2. Both views logged in
next_prompt.md. **No decision yet.** Once decided, lock the choice here
so future rewrites don't re-litigate it.

### Q2 — Larger-dataset ablation
Open. Decision should be made before starting Section 4 rewrite, since the
new CI column depends on which dataset's results we report.
```

### `progress.md`

```markdown
## 2026-04-18

- Round-2 review received from advisor + Reviewer 2.
- 5 must-change items unblocked; 2 questions blocking; 1 backlog idea.
- No paper text edited yet.
- Blocking decisions to make next: framing (Q1), ablation scope (Q2).
```

### `changelog.md`

```markdown
## 2026-04-18

- Reviewed paper v3; processed advisor + Reviewer 2 comments.
- Updated task_plan.md (+5 tasks, 2 decision tasks, 1 backlog).
- Logged unresolved methodology questions in findings.md.
- Generated next_prompt.md with conflicting reviews surfaced explicitly.
- archive/feedback-2026-04-09.md created from prior feedback.md.
```

## 5. Git review

```
$ git status --short
 M task_plan.md
 M findings.md
 M progress.md
 M changelog.md
?? feedback.md
?? next_prompt.md
?? archive/feedback-2026-04-09.md
```

The agent's reply to the user mentions:

- 6 files updated.
- 2 unresolved decisions (framed as Q1 and Q2 in findings.md).
- No paper text edited; awaiting decisions on Q1 and Q2 before rewriting Section 3 and Section 4.
- Suggested next message: *"Use option (c) for Q1, and run the ablation for Q2."* — once that arrives, the agent can carry the next-iteration prompt forward into actual LaTeX edits.

---

> **Takeaway** — long, mixed-tone academic feedback (including a
> reviewer-vs-advisor conflict) becomes a clear set of additive plan changes,
> two clearly framed open decisions, and an audit trail. No premature
> resolution; no lost source material.
