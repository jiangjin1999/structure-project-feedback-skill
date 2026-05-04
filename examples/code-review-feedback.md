# Example 2 — Code review feedback

[English](code-review-feedback.md) | [简体中文](zh-CN/code-review-feedback.md)

> Scenario: a developer just opened PR #142 ("add billing webhook handler").
> Two reviewers leave a mix of corrections, nits, larger redesign hints,
> and a deferral. The developer pastes all comments into the agent.

---

## 1. Raw human feedback (verbatim)

> "Reviewer A: the webhook handler is doing too much in one function — please
> split into `parse → verify → dispatch`. Also missing tests for the signature
> verification path. Nit: `eventType` should be camelCase, not `EventType`.
>
> Reviewer B: looks fine overall. Two things — (1) we should add idempotency
> using the event id, and (2) the structured logger is fine but the log line
> on line 87 leaks the customer email. Optional: maybe extract this whole module
> into a separate package later, but don't do it in this PR.
>
> Also the CI is red, the lint job. I haven't dug into it."

## 2. What the skill does, step by step

### Step 1 — Orient

```
$ git status --short
 M src/webhooks/billing.ts
 M src/webhooks/billing.test.ts

$ gh pr view 142 --json statusCheckRollup --jq '.statusCheckRollup[].conclusion'
SUCCESS
SUCCESS
FAILURE
SUCCESS
```

Skill reads `task_plan.md`, `progress.md`, then opens `billing.ts`.

### Step 2 — Preserve

`feedback.md` gets a verbatim section under `## 2026-04-15 — PR #142 review`.

### Step 3 — Classify

| Bucket | Content |
|---|---|
| **Must change now** | Split handler into `parse → verify → dispatch`. Add tests for signature verification. Fix lint error in CI. Redact customer email from log line 87. Add idempotency keyed by event id. Rename `EventType` → `eventType`. |
| **Keep stable** | Existing structured-logger choice (only the leaky line is a problem). |
| **Intent** | Defensive billing handler with auditability and idempotency. |
| **Evidence** | CI lint job failure (need to read the log). Reviewer A: structure smell. Reviewer B: PII leak. |
| **Constraint** | Idempotency required. PII must not appear in logs. |
| **Question** | None blocking — comments are unambiguous. |
| **Backlog** | Extract whole module to a separate package (deferred to a future PR). |

## 3. Generated `next_prompt.md`

```markdown
# Next Iteration Prompt — PR #142 (billing webhook)

## Context
PR #142 received reviews on 2026-04-15 from two reviewers. CI lint job failing.

## Must Change Now
1. Refactor `handle(req)` into three pure functions:
   - `parse(req): WebhookEvent`
   - `verify(event, signature, secret): VerifiedEvent | SignatureError`
   - `dispatch(event): Promise<void>`
   The top-level `handle` becomes a 5–10 line orchestrator.
2. Add tests in `billing.test.ts` for `verify`:
   - Valid signature passes.
   - Tampered payload fails.
   - Wrong secret fails.
   - Missing signature header fails.
3. Add idempotency: persist `event.id` as a unique key before dispatch; on
   duplicate, return 200 OK without dispatching.
4. Redact customer email in the log line at `billing.ts:87` (use existing
   `redactPII` helper or add `email: maskEmail(event.customer.email)`).
5. Rename `EventType` to `eventType` everywhere (interface field + call sites).
6. Fix the failing lint job in CI; do not silence the rule unless the rule
   itself is wrong.

## Keep Stable
- Structured logger choice and configuration.
- Public route path and HTTP method.
- The webhook secret env var name.

## Constraints
- No PII (email, full card number, billing address) in any log line.
- Idempotency must survive process restart (use the existing DB, not memory).
- All new code typed; no `any`.

## Backlog (do not implement this round)
- Extracting the entire billing-webhook module into a separate package.

## Acceptance Checks
- `pnpm test src/webhooks/billing.test.ts` passes with the four new cases.
- `pnpm lint` passes (CI lint job green).
- Replay the same `event.id` twice in a local script — second call returns 200, no DB write, no dispatch side-effect.
- `rg -n "customer.email" src/webhooks/billing.ts` shows no unredacted use in log calls.
- `EventType` no longer appears anywhere (`rg -n "EventType" src/`).
```

## 4. Updated planning files (additive only)

### `task_plan.md` (diff view)

```diff
 ## PR #142 — billing webhook
-- [ ] Open PR
+- [x] Open PR
+- [ ] Refactor `handle` into parse / verify / dispatch
+- [ ] Add tests for signature verification (4 cases)
+- [ ] Add idempotency keyed by event.id
+- [ ] Redact customer email in log line 87
+- [ ] Rename EventType → eventType
+- [ ] Fix CI lint failure

 ## Backlog
+- [ ] Extract billing-webhook module into a separate package (post-#142)
```

### `findings.md` (excerpt)

```markdown
## 2026-04-15 — Decisions on billing webhook

- **Decision**: Use the existing DB-backed unique key for idempotency.
  Memory or in-process cache rejected — we have multi-instance deploys.
- **Constraint locked in**: No PII in log output. Always pipe through `redactPII`
  helper for any field originating from `event.customer.*`.
- **Style**: Webhook handler module pattern is `parse → verify → dispatch` with
  the entry point staying thin. To be applied to future webhook handlers too.
```

### `progress.md`

```markdown
## 2026-04-15

- PR #142 reviewed; 6 must-change items, 1 deferred.
- CI: 1 lint failure to address; tests currently pass but coverage incomplete.
- Status: ready to implement; no blocking questions.
```

### `changelog.md`

```markdown
## 2026-04-15

- PR #142 review processed; updated task_plan.md (+6 tasks, 1 backlog).
- New decisions recorded in findings.md (idempotency, PII policy, handler shape).
- next_prompt.md regenerated with acceptance checks.
- No code changes yet.
```

## 5. Git review

```
$ git status --short
 M task_plan.md
 M progress.md
 M findings.md
 M changelog.md
?? feedback.md
?? next_prompt.md

$ git diff --stat
 changelog.md   |  6 ++++++
 findings.md    |  9 +++++++++
 progress.md    |  5 +++++
 task_plan.md   | 10 ++++++++--
 4 files changed, 28 insertions(+), 2 deletions(-)
```

After the user says "go", the agent executes the next-iteration prompt
top-to-bottom, runs `pnpm test` and `pnpm lint`, then commits with a message
like:

```
fix(billing): refactor handler, add idempotency, redact PII, fix lint

Closes review threads on PR #142. See findings.md for decisions.
```

---

> **Takeaway** — what looked like a mix of nits, structural rework, and a
> CI failure has been turned into an ordered checklist with explicit
> acceptance commands a reviewer can run before re-approving.
