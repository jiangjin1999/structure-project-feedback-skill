# Iteration 1 — RFC triage from a GitHub Discussion

[English](iteration-1-rfc-triage.md) | [简体中文](zh-CN/iteration-1-rfc-triage.md)

```
┌────────────────────────────────────────────────────────────────────┐
│ ▶ Iteration 1 (you are here) ─→ Iteration 2 ─→ Iteration 3        │
│   RFC triage                    PR review       post-launch        │
└────────────────────────────────────────────────────────────────────┘
```

| Field | Value |
|---|---|
| Host project | [`openai/codex`](https://github.com/openai/codex) (Apache-2.0, ~80k★) |
| Trigger | A 12-comment GitHub Discussion proposing **persistent agent memory across `codex` sessions** |
| Maintainer's goal | Turn the messy thread into an actionable RFC and an open-decisions log |
| State before this iteration | The host repo already exists (it has `README.md`, `AGENTS.md`, `Cargo.toml`, `src/`, `rfcs/`, etc.) but **no plan files yet** — Step 0 (Bootstrap) will run |

---

## 1. Raw human feedback (verbatim, condensed from the discussion)

> **@u_alice** — "I'd love memory across sessions. Right now every `codex` run forgets what AGENTS.md decisions I approved last time. Even just a `~/.codex/memory.json` would be huge."
>
> **@u_bob** — "+1, but please don't use a JSON file. SQLite or sled. JSON gets corrupted with concurrent writes when I run codex in two terminals."
>
> **@u_carol** — "Agree with @u_bob. Also: memory should be **per project**, not global. I have ~30 repos with very different AGENTS.md."
>
> **@u_dan** — "I want it global by default with per-project override. Otherwise onboarding is painful."
>
> **@u_eve** — "Whatever you do, please make sure approvals (the `--approval` mode) are remembered too. Re-typing 'allow' every session is the actual pain."
>
> **@u_frank** — "What happens to memory under sandbox mode? Does the sandbox break the file write?"
>
> **@u_grace** — "Should this be a separate `codex memory` subcommand or just transparent? I'd prefer transparent + a `codex memory list` for inspection."
>
> **@u_henry** — "Off-topic but: please also add multi-modal memory (images from screenshots) someday."
>
> **@u_ivy** — "I'd want encryption at rest. Some of these decisions reference repo paths I'd rather not leak."
>
> **@u_jack** — "Use case: I run a sandbox-restricted codex session for a client. Memory MUST NOT cross between client A and client B."
>
> **@u_kate** — "Maintainer note: this overlaps with discussion #4081 (skills cache). Probably one storage layer for both."
>
> **@u_leo** — "Please ship behind a flag for at least one minor version. Some of us script around codex's stateless behavior."

## 2. What the skill does

### Step 0 — Bootstrap from existing project (first time only)

`codex` is already a real, populated repo. The skill must adapt to it
rather than create a generic `task_plan.md` / `findings.md` / … in the
project root and pollute it.

```bash
$ git ls-files | head -20
.github/workflows/ci.yml
AGENTS.md
Cargo.lock
Cargo.toml
LICENSE
README.md
docs/contributing.md
docs/install.md
rfcs/.keep
src/main.rs
...
```

The agent runs the bootstrap step:

1. **Scan project shape** — sees a Rust workspace (`Cargo.toml`,
   `src/`), an existing `rfcs/` directory, an `AGENTS.md`, and `docs/`.
2. **Map skill files to existing conventions** — there is no
   `task_plan.md`, `findings.md`, `progress.md`, or `changelog.md` at
   the root. There is a `rfcs/` directory that is the natural home for
   feature-level work; the maintainer chooses to scope this RFC's plan
   files into `rfcs/0001-persistent-memory/` instead of polluting the
   root.
3. **Infer the project type** — Rust CLI coding agent.
4. **Record the auto-detected mapping** in the new
   `rfcs/0001-persistent-memory/findings.md`:

   ```markdown
   ## Project shape — auto-detected (2026-04-22)
   - Project type: Rust CLI coding agent (openai/codex).
   - File-kit mapping (scoped to this RFC):
     - feedback    → (new) rfcs/0001-persistent-memory/feedback.md
     - task_plan   → (new) rfcs/0001-persistent-memory/task_plan.md
     - findings    → (new) rfcs/0001-persistent-memory/findings.md
     - progress    → (new) rfcs/0001-persistent-memory/progress.md
     - next_prompt → (new) rfcs/0001-persistent-memory/next_prompt.md
     - changelog   → (new) rfcs/0001-persistent-memory/changelog.md
   - Naming style: lowercase_with_underscores.md (matches docs/).
   - Commit message style: conventional (matches `git log`).
   - Language of prior notes: en.
   - Read-only references: AGENTS.md, README.md, docs/contributing.md.
   - Unsure about: nothing critical; proceeding without a question.
   ```

The bootstrap mapping is the idempotency marker — Iterations 2 and 3
will detect this block and skip Step 0.

> All file paths below are inside `rfcs/0001-persistent-memory/` per
> the mapping above. Paths are written as bare names (`task_plan.md`)
> for readability.

### Step 1 — Orient

```bash
$ git status --short
?? rfcs/0001-persistent-memory/
```

The agent reads `AGENTS.md` (the host project's contract) and notes
that `codex` already has approval modes, sandbox, and a skills cache
referenced by @u_kate. No existing `task_plan.md` etc. to merge with —
the bootstrap mapping above will guide all writes.

### Step 2 — Preserve

`feedback.md` is created and the 12 verbatim comments are pasted under
`## 2026-04-22 — Discussion: persistent agent memory`. Author handles
are kept (`@u_alice`, etc.) so attribution survives summarization.

### Step 3 — Classify

| Bucket | Content |
|---|---|
| **Must change now** | Nothing yet — this is RFC stage, not implementation. |
| **Keep stable** | Existing approval-mode CLI flags (`--approval`); existing sandbox semantics. |
| **Intent** | Reduce repeat configuration cost across sessions; preserve per-project isolation; respect sandbox boundaries. |
| **Evidence** | Two real overlaps: skills cache (#4081 per @u_kate); AGENTS.md decisions; approval-mode UX. |
| **Constraint** | Must work under sandbox. Must not leak across project boundaries. Must be opt-in for at least one minor version (@u_leo). |
| **Question** | (a) Storage backend: JSON vs SQLite vs sled. (b) Scope default: global vs per-project. (c) Subcommand surface vs transparent. (d) Encryption at rest scope. |
| **Backlog** | Multi-modal memory (@u_henry). Cross-tenant guarantees beyond per-project isolation (@u_jack — partially in scope; full multi-tenant deferred). |

### Step 4 — Convert into a `next_prompt.md` (the RFC drafting prompt)

```markdown
# Next-Iteration Prompt — RFC: Persistent Agent Memory

## Context
Discussion (12 comments) on persistent memory across `codex` sessions.
Goal of this iteration: produce a 1-page RFC with explicit open
decisions, ready for a maintainer review meeting.

## Must Change Now
1. Draft `rfcs/0001-persistent-memory.md` covering:
   - Problem statement (cite @u_alice, @u_eve, @u_carol).
   - Non-goals (multi-modal memory; full multi-tenant isolation).
   - Three open design decisions, each with at least 2 options and
     trade-offs (see Open Questions below).
   - Sandbox interaction: behavior under read-only / restricted modes.
   - Migration / opt-in plan: feature flag for ≥ 1 minor version.
2. Cross-reference discussion #4081 (skills cache) and propose a unified
   storage layer where reasonable.

## Keep Stable
- Existing `--approval` CLI flag surface.
- Sandbox boundary semantics (no new escape hatches).
- AGENTS.md spec — memory is consumed alongside, not in place of.

## Constraints
- Memory writes must succeed (or fail explicitly) under sandbox mode.
- Per-project isolation is a hard requirement before global default.
- Opt-in flag for at least one minor version before default-on.

## Open Questions (resolve before implementation)

### Q1 — Storage backend
- (a) JSON file (simplest, but corruption under concurrent writes).
- (b) SQLite (durable, supports queries; +1 dep).
- (c) sled (Rust-native; smaller surface than SQLite).
- Recommend (b) SQLite for ecosystem maturity; revisit if dep weight is contested.

### Q2 — Scope default
- (a) Global by default with per-project override.
- (b) Per-project only; explicit opt-in for global.
- Hard constraint already locked: per-project isolation must work.
- Decision needed before writing the migration plan.

### Q3 — Subcommand surface
- (a) Transparent (read/write happens automatically; `codex memory list` for inspection).
- (b) Explicit `codex memory <get|set|clear>` subcommand only.
- (c) Both.

### Q4 — Encryption at rest
- In scope: per-entry opt-in with OS keychain.
- Out of scope (this RFC): full disk encryption guarantees.

## Backlog (do not work on this round)
- Multi-modal memory (images / screenshots).
- Cross-tenant guarantees beyond per-project isolation.

## Acceptance Checks
- `rfcs/0001-persistent-memory.md` exists, ≤ 2 pages.
- Every Open Question has at least 2 options and a recommendation (or
  an explicit "no recommendation yet").
- Sandbox interaction section names the specific approval modes
  (`--approval auto`, `on-failure`, `untrusted`) and what each does.
- Discussion #4081 explicitly cited.
```

### Step 5 — Update planning files (created from scratch)

`task_plan.md`:

```markdown
# task_plan.md

## Epic — Persistent Agent Memory

### In progress
- [ ] Draft RFC `rfcs/0001-persistent-memory.md`

### Open decisions (see findings.md)
- [ ] Q1 storage backend
- [ ] Q2 scope default
- [ ] Q3 subcommand surface
- [ ] Q4 encryption at rest scope

### Backlog
- [ ] Multi-modal memory
- [ ] Cross-tenant guarantees beyond per-project isolation
```

`findings.md`:

```markdown
# findings.md

## 2026-04-22 — Discussion → RFC stage

### Locked-in constraints
- Memory must work under all sandbox modes; failure must be explicit.
- Per-project isolation is required before any global default.
- Behavior must be opt-in for ≥ 1 minor version.

### Open decisions (mirror task_plan.md)
- Q1 storage backend (JSON / SQLite / sled). Tentative: SQLite.
- Q2 scope default (global+override / per-project only). Open.
- Q3 subcommand surface (transparent / explicit / both). Open.
- Q4 encryption at rest scope: per-entry opt-in with OS keychain in scope; full-disk-level OOS.

### Cross-references
- Discussion #4081 (skills cache) — likely shares a storage layer.
- AGENTS.md spec — memory is additive, not a replacement.
```

`progress.md`:

```markdown
# progress.md

## 2026-04-22 — Iteration 1
- Discussion triage complete; 12 comments classified.
- RFC drafting unblocked; 4 open decisions logged.
- No code changes yet.
- Next milestone: maintainer review of RFC draft.
```

`changelog.md`:

```markdown
# changelog.md

## 2026-04-22 — Iteration 1: RFC triage
- Created `feedback.md` (12 community comments preserved verbatim).
- Created `task_plan.md`, `findings.md`, `progress.md`, `changelog.md`.
- Generated `next_prompt.md` for RFC drafting.
- No source code modified.
```

## 3. Git review

```
$ git status --short
?? changelog.md
?? feedback.md
?? findings.md
?? next_prompt.md
?? progress.md
?? task_plan.md

$ git diff --stat HEAD
 changelog.md    |  6 ++++
 feedback.md     | 28 ++++++++++++++
 findings.md     | 22 ++++++++++++
 next_prompt.md  | 65 +++++++++++++++++++++++++++++++++
 progress.md     |  6 ++++
 task_plan.md    | 16 ++++++++
 6 files changed, 143 insertions(+)
```

The agent's wrap-up message:

> 12 comments preserved in `feedback.md`. RFC drafting unblocked, but
> 4 open decisions are logged in `findings.md` (Q1–Q4). Recommended
> next step: hold a 30-minute maintainer review focused on Q2 (scope
> default), since the migration plan depends on it.

## 4. State carried into Iteration 2

When Iteration 2 starts, **none of these files are thrown away**.
The next agent (or the next session) will read:

- `findings.md` — to know which constraints are locked in.
- `task_plan.md` — to know the open decisions.
- `feedback.md` — to know the original wording (and quote it back if a
  reviewer challenges a decision).

The `archive/` directory is empty in this iteration. It will gain
`archive/feedback-iter-1.md` only if Iteration 2 needs to compress
`feedback.md` to keep it short. We will see exactly that next.

➡ Continue: [Iteration 2 — PR review](iteration-2-pr-review.md)
