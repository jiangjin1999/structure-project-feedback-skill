# 示例 2 —— 代码 PR review 反馈

[English](../code-review-feedback.md) | [简体中文](code-review-feedback.md)

> 场景：开发者刚开了 PR #142（"add billing webhook handler"）。两位 reviewer
> 留下了一堆混合反馈：修正 + nit + 较大的重构建议 + 一项延后建议。
> 开发者把所有评论一次性贴给了智能体。

---

## 1. 原始反馈（按原话保留）

> "Reviewer A：这个 webhook handler 一个函数干太多事了 —— 拆成
> `parse → verify → dispatch`。还缺签名验证路径的测试。Nit：`eventType`
> 应该是 camelCase，不是 `EventType`。
>
> Reviewer B：整体看着还行。两个事 —— (1) 应该用 event id 加一下幂等性，
> (2) structured logger 选型 OK，但第 87 行那条日志泄露了 customer 邮箱。
> Optional：以后可以把整个模块抽成独立 package，但不要在这个 PR 里做。
>
> 还有 CI 是红的，lint job 挂了。我没去看具体原因。"

## 2. skill 一步步做了什么

### Step 1 —— 定位

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

skill 读取 `task_plan.md`、`progress.md`，然后打开 `billing.ts`。

### Step 2 —— 保留

`feedback.md` 里加上 `## 2026-04-15 —— PR #142 review` 一节，原话保留。

### Step 3 —— 分类

| 类别 | 内容 |
|---|---|
| **必须立刻改** | 把 handler 拆成 `parse → verify → dispatch`。补签名验证测试。修 CI 的 lint 失败。第 87 行日志的邮箱字段做脱敏。基于 event id 加幂等性。`EventType` → `eventType`。 |
| **保持稳定** | 现有 structured logger 的选型（只有泄露的那一行有问题）。 |
| **意图** | 一个有审计能力、幂等的 billing webhook handler。 |
| **证据** | CI lint 失败（需读日志）。Reviewer A：结构 smell。Reviewer B：PII 泄露。 |
| **约束** | 必须幂等。日志中不能含 PII。 |
| **待澄清问题** | 没有阻塞性问题 —— 评论都很明确。 |
| **待办池** | 把整个模块抽成独立 package（延后到后续 PR）。 |

## 3. 产出的 `next_prompt.md`

```markdown
# 下一轮 Prompt —— PR #142（billing webhook）

## 上下文
PR #142 在 2026-04-15 收到两位 reviewer 评论。CI lint job 当前为红。

## 必须立刻改
1. 把 `handle(req)` 重构为三个纯函数：
   - `parse(req): WebhookEvent`
   - `verify(event, signature, secret): VerifiedEvent | SignatureError`
   - `dispatch(event): Promise<void>`
   顶层 `handle` 退化为 5–10 行的 orchestrator。
2. 在 `billing.test.ts` 补充 `verify` 的测试：
   - 合法签名通过。
   - payload 被篡改时失败。
   - 错误的 secret 时失败。
   - 缺失签名 header 时失败。
3. 加幂等性：dispatch 之前把 `event.id` 写入唯一键；重复事件直接返回 200 OK，不再 dispatch。
4. 在 `billing.ts:87` 对 customer 邮箱做脱敏（用现有 `redactPII` helper，
   或 `email: maskEmail(event.customer.email)`）。
5. 把 `EventType` 改名为 `eventType`（接口字段 + 所有调用点）。
6. 修复 CI 的 lint failure；除非规则本身错了，否则不要 silence 这条规则。

## 保持稳定
- structured logger 的选型和配置。
- webhook 路由路径和 HTTP 方法。
- webhook secret 的环境变量名。

## 约束
- 任何日志行都不能出现 PII（邮箱、完整卡号、账单地址）。
- 幂等性必须能在进程重启后保持（用现有 DB，不要用内存）。
- 新代码必须有完整类型；不允许 `any`。

## 待办池（本轮不做）
- 把整个 billing-webhook 模块抽成独立 package。

## 验收标准
- `pnpm test src/webhooks/billing.test.ts` 通过，含 4 条新用例。
- `pnpm lint` 通过（CI lint job 转绿）。
- 用本地脚本重放同一个 `event.id` 两次 —— 第二次返回 200，无 DB 写入，无 dispatch 副作用。
- `rg -n "customer.email" src/webhooks/billing.ts` 中没有未脱敏的日志调用。
- `rg -n "EventType" src/` 不再有命中。
```

## 4. 更新计划文件（仅增量）

### `task_plan.md`（diff 视图）

```diff
 ## PR #142 —— billing webhook
-- [ ] 开 PR
+- [x] 开 PR
+- [ ] 把 `handle` 重构为 parse / verify / dispatch
+- [ ] 补签名验证测试（4 条）
+- [ ] 加基于 event.id 的幂等性
+- [ ] 第 87 行日志脱敏 customer 邮箱
+- [ ] EventType → eventType 改名
+- [ ] 修 CI lint 失败

 ## Backlog
+- [ ] 把 billing-webhook 模块抽成独立 package（延后到 #142 之后）
```

### `findings.md`（节选）

```markdown
## 2026-04-15 —— Billing webhook 决策记录

- **决定**：用现有 DB 唯一键做幂等性。否决了内存 / 进程内 cache 方案 ——
  线上是多实例部署。
- **约束锁定**：日志中不允许 PII。任何来自 `event.customer.*` 的字段都
  必须经过 `redactPII` helper。
- **风格**：webhook handler 模块统一为 `parse → verify → dispatch`，入口
  保持薄。后续 webhook handler 也按同一形态。
```

### `progress.md`

```markdown
## 2026-04-15

- PR #142 已 review；6 项必改、1 项延后。
- CI：lint 失败 1 处需修；测试当前通过但覆盖度不足。
- 状态：可以开始实施；无阻塞问题。
```

### `changelog.md`

```markdown
## 2026-04-15

- 处理 PR #142 review；更新 task_plan.md（+6 项任务，+1 backlog）。
- findings.md 记录新决策（幂等性、PII 策略、handler 形态）。
- 重新生成 next_prompt.md，含验收标准。
- 暂未修改代码。
```

## 5. Git 复审

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

用户说 "go" 之后，智能体按 next_prompt.md 自上而下执行，跑 `pnpm test`
和 `pnpm lint`，然后用如下 commit message 提交：

```
fix(billing): refactor handler, add idempotency, redact PII, fix lint

Closes review threads on PR #142. See findings.md for decisions.
```

---

> **要点** —— 看似一团 nits + 结构性重构 + CI 失败的混合反馈，被整理成一份
> 有序的 checklist，并附上 reviewer 在重新 approve 之前可以直接执行的验收命令。
