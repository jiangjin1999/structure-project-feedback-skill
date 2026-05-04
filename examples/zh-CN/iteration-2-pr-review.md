# Iteration 2 —— PR review（含对前一轮决策的挑战）

[English](../iteration-2-pr-review.md) | [简体中文](iteration-2-pr-review.md)

```
┌────────────────────────────────────────────────────────────────────┐
│   Iteration 1 ─→ ▶ Iteration 2（你在这里）─→ Iteration 3            │
│   RFC triage      PR review                    上线后反馈          │
└────────────────────────────────────────────────────────────────────┘
```

| 字段 | 内容 |
|---|---|
| 案例项目 | [`openai/codex`](https://github.com/openai/codex) |
| 触发点 | PR `#PR-4123`（示意 ID），实施 Iteration 1 的 RFC |
| 来自 Iteration 1 的输入 | `findings.md`（Q1–Q4 + 已锁定约束）、`task_plan.md`、`next_prompt.md` |
| 维护者目标 | 处理两位 reviewer 的混合评论，其中一条 **挑战了 Iteration 1 关于 Q1（存储后端）的决策** |

---

## 0. agent 在动手前先读什么（memory replay）

```bash
$ ls -1
AGENTS.md
archive/
changelog.md
feedback.md
findings.md
next_prompt.md
progress.md
rfcs/
task_plan.md
```

agent 在动任何东西之前，先重新读：

- `findings.md` —— 回忆 Iteration 1 **已锁定的约束**（sandbox-safe；
  按项目隔离；至少 1 个 minor version 内 opt-in）以及 Q1 的 **暂定决策**
  （SQLite）。
- `task_plan.md` —— 看哪些任务正在进行（Iteration 1.5 的时候
  `Draft RFC` 已被标记完成；增加了实施任务）。
- 上一轮的 `next_prompt.md` —— 看实施需要满足的验收标准。

这种显式的"memory replay"才是让 Iteration 2 不会重新决策已经定下来的东西
的关键机制。

---

## 1. 原始反馈（按原话保留，来自 PR #PR-4123）

> **Reviewer A（维护者）**："整体不错。几个事：
> 1. 新的 `memory` 模块应该放在 `core/` 而不是 `agent/`，因为 skills cache
>    （PR #PR-4081）也在往 `core/` 迁。
> 2. `memory::flush()` 里有 race condition —— 同一个项目根目录下并发跑两个
>    `codex` 会互相覆盖。需要在测试里复现。
> 3. Nit：`MemEntry` 改名为 `MemoryEntry`，与代码库其它地方风格一致。
> 4. CI 红了，`cargo clippy` 那个 job。看着像一个未使用的 import。"
>
> **Reviewer B（社区贡献者）**："两个想法：
> 1. **我觉得我们应该重新考虑 RFC 里的 Q1。** SQLite 在 Windows 上的依赖
>    树意外地大。我刚踩到 issue #ISS-4099 —— `rusqlite` 在最小化 Windows
>    runner 上构建失败。也许 sled（甚至 flat-file + 文件锁）值得 revisit。
>    我不 block，但请记一下。
> 2. 一个新 backlog 点子：`codex memory export`，方便用户从一个仓库 seed
>    另一个仓库。这个 PR 里别做。"

## 2. skill 一步步做了什么

### Step 1 —— 定位（memory replay）

agent 确认：

- Q1（存储后端）在 `findings.md` 2026-04-22 中是 **暂定 SQLite**。
- 已锁定约束：行为必须在 sandbox 下工作。
- race condition 修复与 **按项目隔离** 这条已锁定约束有交互：修复方案
  不能削弱隔离。

### Step 2 —— 保留

`feedback.md` 追加 PR 评论的原话，置于
`## 2026-05-08 —— PR #PR-4123 review` 一节下。

之前的 `feedback.md`（Iteration 1 的讨论）现在已经很长了。agent 应用自己
的"保留"规则：在做任何缩减 **之前**，把整个旧 `feedback.md` 整体复制到
`archive/feedback-iter-1.md`，确保 Iteration 1 的原话仍然可恢复。

### Step 3 —— 分类

| 类别 | 内容 |
|---|---|
| **必须立刻改** | `memory` 模块挪到 `core/`。修 `flush()` 的 race condition + 回归测试。`MemEntry` 改 `MemoryEntry`。修 `cargo clippy` 失败。 |
| **保持稳定** | sandbox 语义。按项目隔离。本 PR 引入的公共 CLI 表面。 |
| **意图** | 在不削弱任何已锁定约束的前提下，把功能落到 opt-in flag 后面。 |
| **证据（对前一轮决策的挑战）** | Reviewer B 报告 `rusqlite` 在最小化 Windows runner 上构建失败（#ISS-4099）。 |
| **约束** | race-condition 修复不得削弱按项目隔离（来自 Iteration 1 的锁定）。 |
| **待决问题** | **Q1 被挑战。** 重新打开存储后端决策？下方重新评估选项。 |
| **待办池** | `codex memory export`（Reviewer B）。 |

> **关键**：agent **不会** 偷偷翻转 Q1。它在 `findings.md` 里把 Q1 重新打开，
> 附上新证据，并在 `next_prompt.md` 里显式提出来。Iteration 1 的暂定决策
> 不会被抹掉，而是 **被新证据标注 + 附上重新决策请求**。

### Step 4 —— `next_prompt.md`（重新生成）

```markdown
# 下一轮 Prompt —— PR #PR-4123 修复

## 上下文
PR #PR-4123 实施了 `rfcs/0001-persistent-memory.md` 的 RFC。
两位 reviewer 在 2026-05-08 给出评论。CI 红。

## 必须立刻改
1. 把 `src/agent/memory/` 挪到 `src/core/memory/`。更新所有 import。
2. 修 `memory::flush()` 的 race condition：
   - 在写入周围加按项目根目录的 advisory file lock。
   - 加回归测试：起两个 writer，断言 last-writer-wins 语义且不损坏文件。
   - 不得削弱按项目隔离（见 findings.md 2026-04-22）。
3. 把 `MemEntry` 改名为 `MemoryEntry`（struct + 所有用法）。
4. 修复 `cargo clippy` 失败（很可能是未使用的 import；除非规则本身错了，
   否则不要 silence 警告）。

## 保持稳定
- sandbox 交互层。
- 按项目隔离（已锁定约束）。
- 本 PR 引入的公共 CLI 表面。

## 待决问题

### Q1-revisited —— 存储后端（被 Reviewer B 重新打开）
- 原决策（2026-04-22）：暂定 SQLite。
- 新证据（2026-05-08）：`rusqlite` 在最小化 Windows CI runner 上构建失败，
  阻塞 issue #ISS-4099。
- 本轮要评估的选项：
  (a) 留 SQLite；显式文档 + CI 测试最小化 Windows runner。
  (b) 切换到 sled（依赖更小，Rust 原生）。
  (c) flat-file + 文件锁（与新的 race-fix 思路对齐）。
- **合并前必须决定。** 推荐：(c) —— 与 Must Change #2 的 file-lock
  思路对齐，是与 race-fix 风险最低的匹配。

## 待办池（本轮不做）
- `codex memory export` 子命令（Reviewer B）。
- Multi-modal memory（从 Iteration 1 carry over）。

## 约束（carry over，仍锁定）
- sandbox-safe 写入；显式失败模式。
- 按项目隔离。
- 至少 1 个 minor version 内 opt-in（尚未默认开启）。

## 验收标准
- `cargo test --workspace` 绿。
- `cargo clippy --all-targets -- -D warnings` 绿。
- 新回归测试能复现原 race，并通过修复后版本。
- `rg -n "agent::memory" .` 返回空（没有遗留引用）。
- `rg -n "MemEntry" .` 返回空。
- 合并前在 `findings.md` 中 **决定** Q1-revisited。
```

### Step 5 —— 增量更新计划文件

`task_plan.md`（相对 Iteration 1 的 diff）：

```diff
 # task_plan.md

 ## Epic —— 持久化 Agent Memory

 ### In progress
-- [ ] 起草 RFC `rfcs/0001-persistent-memory.md`
+- [x] 起草 RFC `rfcs/0001-persistent-memory.md`
+- [ ] memory 模块从 `agent/` 挪到 `core/`
+- [ ] `flush()` 的 race condition 修复（含回归测试）
+- [ ] 重命名 `MemEntry` → `MemoryEntry`
+- [ ] 修复 `cargo clippy` 失败
+- [ ] 重新决定 Q1（存储后端）—— 见 findings.md

 ### 未决决策（见 findings.md）
-- [ ] Q1 存储后端
+- [ ] Q1 存储后端  （2026-05-08 重新打开；新证据 #ISS-4099）
- [ ] Q2 默认 scope
- [ ] Q3 子命令表面
- [ ] Q4 静态加密范围

 ### Backlog
+- [ ] `codex memory export` 子命令
 - [ ] Multi-modal memory
 - [ ] 超出按项目隔离的多租户保证
```

`findings.md`（diff）：

```diff
 # findings.md

 ## 2026-04-22 —— Discussion → RFC 阶段

 ### 已锁定的约束
 ...

 ### 未决决策（与 task_plan.md 同步）
-- Q1 存储后端（JSON / SQLite / sled）。暂定：SQLite。
+- Q1 存储后端（JSON / SQLite / sled / flat+lock）。暂定 SQLite **2026-05-08 已 REVISITED**，见下文。

+## 2026-05-08 —— PR #PR-4123 review
+
+### Q1 重新评估 —— 存储后端
+- 新证据：`rusqlite` 在最小化 Windows CI runner 上构建失败（#ISS-4099）。
+- 重新评估见 next_prompt.md（选项 a/b/c）。合并前必须决定。
+- Iteration 1 的推理 **不会** 被抹掉；保存在 `archive/feedback-iter-1.md`。
+
+### 新增约束
+- race condition 修复必须用与按项目隔离兼容的机制。
+  按项目根目录的 advisory file lock 满足；全局 lock 不满足。
```

`progress.md`：

```markdown
## 2026-05-08 —— Iteration 2
- 实施 PR #PR-4123 收到 Reviewer A + B 评论。
- 4 项必改、1 项重新打开的决策（Q1）、1 项新 backlog。
- archive/ 现含 feedback-iter-1.md（旧 feedback.md 的快照）。
- 状态：实施完成 80%；合并阻塞在 Q1 重新决策上。
```

`changelog.md`：

```markdown
## 2026-05-08 —— Iteration 2: PR #PR-4123 review
- feedback.md 追加 PR review 原话。
- archive/feedback-iter-1.md 从原 feedback.md 创建。
- task_plan.md：+5 任务，Q1 标记为重新打开。
- findings.md：登记 Q1 重新评估；锁定新的 race-fix 约束。
- next_prompt.md 重新生成，含 re-fix 验收标准。
- 暂未修改源代码 —— 实施在 Q1 重新决策之后。
```

## 3. Git 复审

```
$ git status --short
 M task_plan.md
 M findings.md
 M progress.md
 M changelog.md
 M feedback.md
 M next_prompt.md
?? archive/feedback-iter-1.md

$ git diff --stat
 changelog.md                |  9 +++++++
 feedback.md                 | 22 +++++++++++++++
 findings.md                 | 14 ++++++++++
 next_prompt.md              | 56 ++++++++++++++++++++++++++++++++--
 progress.md                 |  7 +++++
 task_plan.md                | 11 ++++++--
 archive/feedback-iter-1.md  | 28 +++++++++++++++++  (新)
 7 files changed, 142 insertions(+), 5 deletions(-)
```

agent 收尾消息：

> Iteration 2 已处理。4 项机械修复已 unblock；合并阻塞在 **Q1-revisited**
> （存储后端）—— 见 `findings.md` 2026-05-08。新证据（#ISS-4099）让选项
> (c) "flat-file + 文件锁" 成为推荐路径，因为它与 race-fix 机制对齐。
> 在动源代码之前，需要 10 分钟的维护者决策。

## 4. 带入 Iteration 3 的状态

经过 Iteration 2，文件式 memory 已经成长：

- `findings.md` 现含 **两个带日期的小节**（`2026-04-22`、`2026-05-08`）。
  旧条目不删除，而是 **被注释**。
- `task_plan.md` 展示每个任务的全生命：Iteration 1 创建、Iteration 2 进行中、
  等一个决策。
- `archive/feedback-iter-1.md` 让未来读者（或下一轮 agent）随时能恢复
  Iteration 1 中任何社区评论的精确措辞。
- `changelog.md` 现在是 2 条目时间线；新读者 20 秒可扫完。

Iteration 3 会依赖这一切。具体来说，当用户上线后提一个 **看起来像
Iteration 1 已经 deferred 过的 multi-modal memory 点子** 的功能请求时，
agent 会先去 `findings.md` 和 `Backlog` 里查 —— 然后才回应。

⬅ 返回：[Iteration 1 —— RFC triage](iteration-1-rfc-triage.md)
➡ 继续：[Iteration 3 —— 上线后反馈](iteration-3-post-launch-feedback.md)
