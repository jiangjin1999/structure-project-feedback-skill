# Iteration 3 — Post-launch user feedback with deferred-idea revival

[English](iteration-3-post-launch-feedback.md) | [简体中文](zh-CN/iteration-3-post-launch-feedback.md)

```
┌────────────────────────────────────────────────────────────────────┐
│   Iteration 1 ─→ Iteration 2 ─→ ▶ Iteration 3 (you are here)       │
│   RFC triage      PR review        post-launch                     │
└────────────────────────────────────────────────────────────────────┘
```

| Field | Value |
|---|---|
| Host project | [`openai/codex`](https://github.com/openai/codex) |
| Trigger | After v0.5 ships, three GitHub issues + one Discord thread land in the same week |
| Inputs from prior iterations | full file kit (`task_plan.md`, `findings.md`, `progress.md`, `changelog.md`, `feedback.md`, `archive/feedback-iter-1.md`) |
| Maintainer's goal | Decide what is a v0.5.1 patch, what is v0.6 work, what is already on file (don't re-litigate), and what conflicts with prior decisions |

---

## 0. What the agent reads first (memory replay)

```bash
$ ls -1 archive/
feedback-iter-1.md
feedback-iter-2.md

$ git log --oneline -- task_plan.md findings.md
abc1234 docs: post-PR-4123 plan update
def5678 docs: re-decide Q1 (flat+lock); race-fix landed
9876543 docs: RFC stage open decisions
```

The agent re-reads, in order:

- `findings.md` — to recall:
  - locked-in constraints (sandbox-safe; per-project isolation; opt-in flag).
  - Q1 final decision after Iteration 2: **flat-file with file-locking**.
  - Q4: **per-entry encryption deferred to follow-up RFC**.
- `task_plan.md` — to see the **Backlog** section (multi-modal memory; encryption follow-up; `codex memory export`).
- `archive/feedback-iter-1.md` — only when Iteration 3 feedback overlaps with comments from the original discussion.
- `changelog.md` — to construct a `v0.4 → v0.5 → v0.5.1` timeline for the user-facing notes.

This replay is what makes Iteration 3 different from "an agent meeting the project for the first time".

---

## 1. Raw human feedback (verbatim, condensed across channels)

> **Issue #ISS-5101 (@u_alice)**: "Loving v0.5! But — `codex memory list` is ~3s on my big monorepo. Was instant in dev builds?"
>
> **Issue #ISS-5102 (@u_mia)**: "Hit a panic on first run after upgrade if `~/.codex/memory/` already existed from a manual experiment I did. Stack trace attached. Loss of UX, but no data corruption."
>
> **Issue #ISS-5103 (@u_henry)**: "Now that memory is here, can we **also store screenshots from approval prompts**? Multi-modal memory! :pray:"
>
> **Discord (@u_pat)**: "Please add encryption at rest. I keep approving file paths I don't want sitting in plaintext on disk."
>
> **Discord (@u_quinn)**: "FYI — I see two `.codex-lock` files in my project root after a crashed session. Cleanup needed?"
>
> **Discord (@u_ray)**: "The race-fix is great, but `flush()` now fails silently when the lock is held >5s. Spinner says 'flushed' even though file unchanged."
>
> **Issue #ISS-5104 (@u_sam)**: "Suggestion: a `codex memory export` would help me bootstrap a new repo from another. Would PR if accepted."

## 2. What the skill does

### Step 1 — Orient (memory replay against findings.md)

The agent maps each comment against existing project memory:

| Comment | Memory match | Decision |
|---|---|---|
| #ISS-5101 perf regression | New — no prior context | Goes to "Must change now" if reproducible. |
| #ISS-5102 panic on dirty `~/.codex/memory/` | New, but constraint check needed | "Sandbox-safe + explicit failure" constraint applies. |
| #ISS-5103 multi-modal memory | **Already in Backlog** since Iteration 1 (`@u_henry` made the same request) | Acknowledge, do NOT re-litigate. |
| Discord encryption at rest | **Already in Backlog** (Q4 deferred to follow-up RFC) | Acknowledge, point to deferred RFC. |
| Discord stale `.codex-lock` files | New — interacts with Iteration 2's race fix | Must change now. |
| Discord silent `flush()` failure | Conflicts with Iteration 1 locked constraint ("explicit failure") | Must change now (constraint violation). |
| #ISS-5104 `codex memory export` | **Already in Backlog** (Reviewer B suggested it in Iteration 2) | Promote to v0.6 epic if reviewer wants it. |

### Step 2 — Preserve

- New verbatim feedback appended to `feedback.md` under `## 2026-05-22 — v0.5 post-launch feedback`.
- `feedback.md` from Iteration 2 (now combined with Iteration 1) is
  snapshotted to `archive/feedback-iter-2.md` before any reduction.

### Step 3 — Classify

| Bucket | Content |
|---|---|
| **Must change now (v0.5.1 patch)** | Perf regression in `memory list` (#ISS-5101). Panic on pre-existing `~/.codex/memory/` (#ISS-5102). Stale lockfile cleanup. Silent `flush()` failure (constraint violation). |
| **Keep stable** | Storage backend (flat+lock, decided in Iter 2). Per-project isolation. Opt-in flag still on. |
| **Intent** | Honor the original RFC's contract; the patch must not change the storage shape. |
| **Evidence** | Stack trace in #ISS-5102; spinner-vs-file-state mismatch reported by @u_ray. |
| **Constraint** | Hot-fix patch — no breaking changes; v0.5.1 must be drop-in. |
| **Question** | None blocking the patch. (Multi-modal memory & encryption RFCs are open but not on this round's path.) |
| **Backlog → v0.6 candidates** | `codex memory export` (now has a willing PR author). Multi-modal memory follow-up RFC. Encryption-at-rest follow-up RFC. |

> **Crucial: the multi-modal memory and encryption requests are not new
> work this round.** They were deferred in Iteration 1 (`findings.md`)
> and the agent's reply explicitly references the existing entry rather
> than spinning up new debate.

### Step 4 — `next_prompt.md` (regenerated for v0.5.1)

```markdown
# Next-Iteration Prompt — v0.5.1 patch release

## Context
v0.5 shipped persistent agent memory on 2026-05-15. One week of user
feedback surfaced four bugs, one constraint violation, and three
already-deferred ideas. This iteration ships a patch; the deferred
ideas remain deferred.

## Must Change Now (v0.5.1)
1. Investigate and fix `codex memory list` perf regression
   (#ISS-5101). Add a benchmark guarding against future regressions.
2. Fix panic on first run when `~/.codex/memory/` already exists
   (#ISS-5102). Migration path: detect, warn, offer `--migrate`
   subcommand.
3. Add stale `.codex-lock` cleanup on startup (when no live PID owns it).
4. Fix silent `flush()` failure on lock timeout. Surface as explicit
   error, matching the locked-in "explicit failure" constraint
   (findings.md 2026-04-22).

## Keep Stable
- Storage backend (flat-file + advisory lock). Decided 2026-05-08, not revisiting.
- Per-project isolation.
- v0.5 public CLI surface (no breaking changes in a patch).

## Acknowledge but DO NOT work this round
- Multi-modal memory (#ISS-5103) → **already in Backlog since Iteration 1.**
  Reply with link to `findings.md` 2026-04-22 and the deferred-RFC plan.
- Encryption at rest (Discord @u_pat) → **Q4 in `findings.md`,
  deferred to follow-up RFC.** Reply with the same.
- `codex memory export` (#ISS-5104) → **In Backlog since Iteration 2.**
  Promote to v0.6 epic if @u_sam wants to PR; do not start now.

## Constraints
- Patch release; no breaking changes.
- "Explicit failure" rule (Iteration 1 locked-in) applies to every
  new error path.

## Acceptance Checks
- `cargo bench memory_list` shows perf within 1.5× of pre-regression.
- Manual repro of #ISS-5102 (pre-existing dir): no panic; clear
  warning; `--migrate` works.
- `.codex-lock` cleanup tested with a killed-process repro.
- `flush()` returns `Err` with a typed variant on lock timeout; UI
  surfaces it (no silent success).
- Three "DO NOT work" items have an issue reply each, citing the
  existing `findings.md` entry by date.
```

### Step 5 — Update planning files (additively)

`task_plan.md` (diff):

```diff
 # task_plan.md

 ## Epic — Persistent Agent Memory

 ### In progress
 ...
+- [ ] v0.5.1 perf fix (#ISS-5101)
+- [ ] v0.5.1 migration for pre-existing memory dir (#ISS-5102)
+- [ ] v0.5.1 stale lockfile cleanup
+- [ ] v0.5.1 explicit failure on flush lock timeout (constraint fix)

 ### Backlog
 - [ ] Multi-modal memory                 (deferred since 2026-04-22)
 - [ ] Cross-tenant guarantees           (deferred since 2026-04-22)
-- [ ] `codex memory export` subcommand
+- [ ] `codex memory export` subcommand   (v0.6 candidate; PR offered by @u_sam)
+- [ ] Encryption-at-rest follow-up RFC   (deferred since 2026-04-22, Q4)
```

`findings.md` (diff):

```diff
 ## 2026-04-22 — Discussion → RFC stage
 ...
 ## 2026-05-08 — PR #PR-4123 review
 ...
+## 2026-05-22 — v0.5 post-launch
+
+### Re-confirmed (no change)
+- Storage backend = flat-file + advisory lock (decided 2026-05-08).
+- Per-project isolation.
+- "Explicit failure" rule applies to all new error paths — used to
+  reject the silent `flush()` behavior reported by @u_ray.
+
+### New constraint (patch-release scope)
+- v0.5.1 is a patch; no breaking CLI or storage-shape changes.
+
+### Re-affirmed deferrals (no new debate)
+- Multi-modal memory: deferred since 2026-04-22 (Backlog).
+- Encryption at rest: Q4, deferred to follow-up RFC.
+- `codex memory export`: deferred since 2026-05-08 (now v0.6 candidate).
```

`progress.md`:

```markdown
## 2026-05-22 — Iteration 3
- v0.5 post-launch triage complete.
- 4 patch items unblocked for v0.5.1; 3 deferred items re-affirmed
  with explicit issue replies linking to findings.md entries.
- archive/ now has feedback-iter-1.md and feedback-iter-2.md.
- Status: v0.5.1 implementation can start; v0.6 epic to be planned
  separately once @u_sam confirms PR intent.
```

`changelog.md`:

```markdown
## 2026-05-22 — Iteration 3: v0.5 post-launch
- 4 v0.5.1 patch tasks added to task_plan.md.
- 3 deferred ideas re-affirmed; explicit issue replies prepared.
- findings.md: re-confirmed Iter 1 + Iter 2 decisions; added v0.5.1
  patch-scope constraint.
- archive/feedback-iter-2.md created.
- No source code modified yet.
```

## 3. Git review

```
$ git status --short
 M task_plan.md
 M findings.md
 M progress.md
 M changelog.md
 M feedback.md
 M next_prompt.md
?? archive/feedback-iter-2.md

$ git log --oneline -- changelog.md
xyz9876 docs: iter 3 post-launch triage
abc1234 docs: post-PR-4123 plan update
def5678 docs: re-decide Q1 (flat+lock); race-fix landed
9876543 docs: RFC stage open decisions
```

## 4. The actual value the file-based memory delivered

Look at the three things this iteration's agent did **not** do:

1. **Did not re-debate multi-modal memory.** It found `@u_henry`'s name
   in `archive/feedback-iter-1.md`, matched it to the Backlog entry
   from Iteration 1, and replied with a link instead of opening a
   parallel design discussion.
2. **Did not re-decide the storage backend.** Even though one comment
   touched lockfile behavior, the agent recognized the storage choice
   was already finalized in Iteration 2 and only fixed the symptoms
   inside that choice.
3. **Did not silently violate the "explicit failure" constraint** when
   handling the lock-timeout report. It reached back to
   `findings.md` 2026-04-22, recognized the violation, and made fixing
   it a "must change now" item — even though no v0.5.1 user demanded
   the constraint by name.

That last one is the strongest demonstration: a constraint set in
Iteration 1 still **actively shaped behavior** in Iteration 3, with
no human re-stating it. That is what file-based memory buys you.

⬅ Back: [Iteration 2 — PR review](iteration-2-pr-review.md)
⏪ Start over: [Iteration 1 — RFC triage](iteration-1-rfc-triage.md)
