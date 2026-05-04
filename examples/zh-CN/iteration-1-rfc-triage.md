# Iteration 1 —— RFC triage（来自 GitHub Discussion）

[English](../iteration-1-rfc-triage.md) | [简体中文](iteration-1-rfc-triage.md)

```
┌────────────────────────────────────────────────────────────────────┐
│ ▶ Iteration 1（你在这里）─→ Iteration 2 ─→ Iteration 3              │
│   RFC triage                  PR review       上线后反馈            │
└────────────────────────────────────────────────────────────────────┘
```

| 字段 | 内容 |
|---|---|
| 案例项目 | [`openai/codex`](https://github.com/openai/codex)（Apache-2.0，~80k★） |
| 触发点 | 一条含 12 条评论的 GitHub Discussion，提议给 `codex` 增加 **跨 session 的持久化 agent memory** |
| 维护者目标 | 把混乱的讨论串转化为可执行的 RFC + 一份未决问题清单 |
| 本轮迭代之前的 file kit 状态 | 空 |

---

## 1. 原始反馈（按原话保留，从讨论中浓缩）

> **@u_alice** —— "我太想要跨 session 的 memory 了。现在每次运行 `codex` 都
> 忘了我上次在 AGENTS.md 里 approve 过哪些决策。哪怕只是一个
> `~/.codex/memory.json` 都好。"
>
> **@u_bob** —— "+1，但请别用 JSON 文件。SQLite 或 sled 都行。我同时开两个
> 终端跑 codex 的时候 JSON 经常被并发写坏。"
>
> **@u_carol** —— "同意 @u_bob。另外：memory 应该是 **按项目** 隔离的，不能是
> 全局的。我有 ~30 个仓库，AGENTS.md 完全不一样。"
>
> **@u_dan** —— "我希望默认全局 + 按项目 override。否则上手成本太高。"
>
> **@u_eve** —— "无论怎么做，请确保 approvals（`--approval` 模式）也被记住。
> 每次重新输 'allow' 才是真正的痛点。"
>
> **@u_frank** —— "在 sandbox 模式下 memory 怎么办？sandbox 会不会让文件
> 写入失败？"
>
> **@u_grace** —— "应该做成单独的 `codex memory` 子命令，还是透明？我倾向
> 透明 + 一个 `codex memory list` 用来检查。"
>
> **@u_henry** —— "跑题但是：以后能不能也加上 multi-modal memory（截图）？"
>
> **@u_ivy** —— "我想要静态加密。这些决策里有些会引用我不太想泄露的仓库路径。"
>
> **@u_jack** —— "use case：我给客户跑 sandbox-restricted 的 codex session。
> memory 绝对不能在客户 A 和客户 B 之间互窜。"
>
> **@u_kate** —— "维护者备注：这跟 discussion #4081（skills cache）有重叠。
> 应该一个 storage layer 同时承担两件事。"
>
> **@u_leo** —— "至少先放在一个 flag 后面，撑过一个 minor version。我们有些
> 人围绕 codex 的无状态行为写了脚本。"

## 2. skill 一步步做了什么

### Step 1 —— 定位

```bash
$ git status --short
?? rfcs/

# 还没有任何计划文件 —— 这是第一轮迭代。
```

智能体先读 `AGENTS.md`（这是案例项目自己的契约），注意到 `codex` 已经有
approval modes、sandbox，并且 @u_kate 提到了 skills cache。没有现成的
`task_plan.md` 等需要合并。

### Step 2 —— 保留

创建 `feedback.md`，把 12 条原话评论粘贴在
`## 2026-04-22 —— Discussion: persistent agent memory` 下。作者 handle
（`@u_alice` 等）保留，这样归纳之后归属仍然可追溯。

### Step 3 —— 分类

| 类别 | 内容 |
|---|---|
| **必须立刻改** | 暂无 —— 这是 RFC 阶段，不是实施阶段。 |
| **保持稳定** | 现有 approval-mode CLI flag（`--approval`）；现有 sandbox 语义。 |
| **意图** | 降低跨 session 的重复配置成本；维持按项目隔离；尊重 sandbox 边界。 |
| **证据** | 两处真实重叠：skills cache（@u_kate 提到的 #4081）；AGENTS.md 决策；approval-mode UX。 |
| **约束** | 必须能在 sandbox 下工作。不能跨项目泄露。至少一个 minor version 内是 opt-in（@u_leo）。 |
| **待决问题** | (a) 存储后端：JSON / SQLite / sled。(b) 默认 scope：全局 / 按项目。(c) 子命令 vs 透明。(d) 静态加密范围。 |
| **待办池** | Multi-modal memory（@u_henry）。超出按项目隔离之外的多租户保证（@u_jack —— 部分在 scope 内；完整多租户延后）。 |

### Step 4 —— 转换为 `next_prompt.md`（RFC 起草 prompt）

```markdown
# 下一轮 Prompt —— RFC: 持久化 Agent Memory

## 上下文
12 条评论的讨论，关于 `codex` 跨 session 的持久化 memory。本轮目标：
产出一份 1 页的 RFC，把所有未决决策显式列出，准备走维护者 review。

## 必须立刻改
1. 起草 `rfcs/0001-persistent-memory.md`，覆盖：
   - 问题陈述（引用 @u_alice、@u_eve、@u_carol）。
   - 非目标（multi-modal memory；完整多租户隔离）。
   - 三个开放设计决策，每个至少 2 个选项 + 权衡（见下方 Open Questions）。
   - sandbox 交互：在 read-only / restricted 模式下的行为。
   - 迁移 / opt-in 计划：feature flag 至少存在 1 个 minor version。
2. 与 discussion #4081（skills cache）交叉引用，并在合理处提议统一 storage layer。

## 保持稳定
- 现有 `--approval` CLI flag 表面。
- sandbox 边界语义（不引入新的 escape hatch）。
- AGENTS.md 规范 —— memory 是叠加在它上面的，不是替代。

## 约束
- memory 写入在 sandbox 模式下必须成功（或显式失败）。
- 在引入全局默认之前，按项目隔离是硬性要求。
- 至少一个 minor version 内是 opt-in flag。

## 待决问题（实施前需要决定）

### Q1 —— 存储后端
- (a) JSON 文件（最简，但并发写易损坏）。
- (b) SQLite（耐用，支持查询；多一个依赖）。
- (c) sled（Rust 原生；表面比 SQLite 小）。
- 推荐：(b) SQLite，生态成熟；如果依赖体积有争议再 revisit。

### Q2 —— 默认 scope
- (a) 默认全局 + 按项目 override。
- (b) 仅按项目；显式 opt-in 走全局。
- 已锁定的硬性约束：按项目隔离必须工作。
- 写迁移计划之前必须决定。

### Q3 —— 子命令表面
- (a) 透明（自动读写；`codex memory list` 用来检查）。
- (b) 仅显式 `codex memory <get|set|clear>` 子命令。
- (c) 两者都要。

### Q4 —— 静态加密
- 在 scope 内：每条目 opt-in，使用 OS keychain。
- 超出 scope（本 RFC 不做）：完整磁盘加密保证。

## 待办池（本轮不做）
- Multi-modal memory（图片 / 截图）。
- 超出按项目隔离的多租户保证。

## 验收标准
- `rfcs/0001-persistent-memory.md` 存在，≤ 2 页。
- 每个 Open Question 至少 2 个选项 + 一个推荐（或显式 "暂无推荐"）。
- sandbox 交互一节明确写出具体的 approval 模式（`--approval auto`、
  `on-failure`、`untrusted`）和各自行为。
- 显式引用 discussion #4081。
```

### Step 5 —— 更新计划文件（从零创建）

`task_plan.md`：

```markdown
# task_plan.md

## Epic —— 持久化 Agent Memory

### In progress
- [ ] 起草 RFC `rfcs/0001-persistent-memory.md`

### 未决决策（见 findings.md）
- [ ] Q1 存储后端
- [ ] Q2 默认 scope
- [ ] Q3 子命令表面
- [ ] Q4 静态加密范围

### Backlog
- [ ] Multi-modal memory
- [ ] 超出按项目隔离的多租户保证
```

`findings.md`：

```markdown
# findings.md

## 2026-04-22 —— Discussion → RFC 阶段

### 已锁定的约束
- memory 在所有 sandbox 模式下必须工作；失败必须是显式的。
- 引入全局默认之前，按项目隔离必须先工作。
- 行为必须在至少 1 个 minor version 内是 opt-in。

### 未决决策（与 task_plan.md 同步）
- Q1 存储后端（JSON / SQLite / sled）。暂定：SQLite。
- Q2 默认 scope（全局+override / 仅按项目）。未决。
- Q3 子命令表面（透明 / 显式 / 都要）。未决。
- Q4 静态加密范围：每条目 opt-in + OS keychain 在 scope 内；
  完整磁盘加密 OOS。

### 交叉引用
- discussion #4081（skills cache）—— 大概率共用一个 storage layer。
- AGENTS.md 规范 —— memory 是叠加，不是替代。
```

`progress.md`：

```markdown
# progress.md

## 2026-04-22 —— Iteration 1
- 讨论 triage 完成；12 条评论已分类。
- RFC 起草已 unblock；4 项未决决策已登记。
- 暂未修改代码。
- 下一里程碑：RFC 草稿走维护者 review。
```

`changelog.md`：

```markdown
# changelog.md

## 2026-04-22 —— Iteration 1: RFC triage
- 创建 `feedback.md`（12 条社区评论按原话保留）。
- 创建 `task_plan.md`、`findings.md`、`progress.md`、`changelog.md`。
- 生成 `next_prompt.md` 用于 RFC 起草。
- 未修改源代码。
```

## 3. Git 复审

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

智能体的收尾消息：

> 12 条评论保留在 `feedback.md`。RFC 起草已 unblock，但有 4 项未决决策
> 登记在 `findings.md`（Q1–Q4）。建议下一步：开一次 30 分钟的维护者
> review，先聚焦 Q2（默认 scope），因为迁移计划依赖于它。

## 4. 带入 Iteration 2 的状态

进入 Iteration 2 时，**这些文件一个都不会被丢弃**。
下一轮的 agent（或下一次 session）会读：

- `findings.md` —— 知道哪些约束已经锁定。
- `task_plan.md` —— 知道哪些是未决决策。
- `feedback.md` —— 知道原话措辞（万一 reviewer 挑战某个决策，可以引回去）。

`archive/` 目录在本轮还是空的。下一轮如果需要把 `feedback.md` 压短，
才会出现 `archive/feedback-iter-1.md`。我们会在下一轮看到正是这件事。

➡ 继续：[Iteration 2 —— PR review](iteration-2-pr-review.md)
