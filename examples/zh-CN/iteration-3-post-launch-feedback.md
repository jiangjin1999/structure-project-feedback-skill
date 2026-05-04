# Iteration 3 —— 上线后用户反馈（含 deferred idea 的复活）

[English](../iteration-3-post-launch-feedback.md) | [简体中文](iteration-3-post-launch-feedback.md)

```
┌────────────────────────────────────────────────────────────────────┐
│   Iteration 1 ─→ Iteration 2 ─→ ▶ Iteration 3（你在这里）           │
│   RFC triage      PR review         上线后反馈                      │
└────────────────────────────────────────────────────────────────────┘
```

| 字段 | 内容 |
|---|---|
| 案例项目 | [`openai/codex`](https://github.com/openai/codex) |
| 触发点 | v0.5 上线后，同一周内涌进 3 条 GitHub issue + 1 条 Discord 帖子 |
| 来自前几轮的输入 | 完整的 file kit（`task_plan.md`、`findings.md`、`progress.md`、`changelog.md`、`feedback.md`、`archive/feedback-iter-1.md`） |
| 维护者目标 | 决定哪些是 v0.5.1 patch、哪些是 v0.6 工作、哪些早就在备忘里（不再争论）、哪些与前几轮决策有冲突 |

---

## 0. agent 在动手前先读什么（memory replay）

```bash
$ ls -1 archive/
feedback-iter-1.md
feedback-iter-2.md

$ git log --oneline -- task_plan.md findings.md
abc1234 docs: post-PR-4123 plan update
def5678 docs: re-decide Q1 (flat+lock); race-fix landed
9876543 docs: RFC stage open decisions
```

agent 按以下顺序重新读：

- `findings.md` —— 回忆：
  - 已锁定约束（sandbox-safe；按项目隔离；opt-in flag）。
  - Q1 在 Iteration 2 之后的最终决策：**flat-file + 文件锁**。
  - Q4：**每条目加密延后到 follow-up RFC**。
- `task_plan.md` —— 看 **Backlog** 一节（multi-modal memory；加密 follow-up；
  `codex memory export`）。
- `archive/feedback-iter-1.md` —— 仅当 Iteration 3 的反馈与原讨论有重叠时
  才打开。
- `changelog.md` —— 为用户面向的 release notes 构建一条
  `v0.4 → v0.5 → v0.5.1` 时间线。

正是这个 replay，让 Iteration 3 区别于"一个第一次接触本项目的 agent"。

---

## 1. 原始反馈（按原话保留，跨渠道浓缩）

> **Issue #ISS-5101（@u_alice）**："喜欢 v0.5！但是 —— `codex memory list`
> 在我的大 monorepo 上要 ~3 秒？dev build 时是瞬时的。"
>
> **Issue #ISS-5102（@u_mia）**："升级后第一次跑就 panic 了，因为我之前手动
> 实验过，`~/.codex/memory/` 已经存在。stack trace 见附件。UX 损失，无数据
> 损坏。"
>
> **Issue #ISS-5103（@u_henry）**："既然 memory 上线了，可以 **顺便存
> approval prompt 的截图** 吗？多模态 memory！🙏"
>
> **Discord（@u_pat）**："请加上静态加密。我不想我 approve 过的文件路径
> 以明文形式留在磁盘上。"
>
> **Discord（@u_quinn）**："FYI —— 一次崩溃 session 之后我项目根目录里
> 看到了两个 `.codex-lock` 文件。需要清理吗？"
>
> **Discord（@u_ray）**："race-fix 很好，但 lock 持有 >5s 时 `flush()` 现在
> 会静默失败。spinner 显示 'flushed'，但文件实际没动。"
>
> **Issue #ISS-5104（@u_sam）**："建议：`codex memory export` 能帮我从一个
> 仓库 bootstrap 另一个。如果接受，我可以 PR。"

## 2. skill 一步步做了什么

### Step 1 —— 定位（针对 findings.md 做 memory replay）

agent 把每条评论与现有项目 memory 做映射：

| 评论 | memory 匹配 | 决策 |
|---|---|---|
| #ISS-5101 性能 regression | 全新 —— 无前置上下文 | 若可复现，进 "必须立刻改"。 |
| #ISS-5102 dirty `~/.codex/memory/` panic | 全新，但需检查约束 | 触发"sandbox-safe + 显式失败"约束。 |
| #ISS-5103 multi-modal memory | **已在 Backlog（自 Iteration 1 起）**（@u_henry 当时就提过） | 回复 + 引用，**不重新争论**。 |
| Discord 静态加密 | **已在 Backlog**（Q4 延后到 follow-up RFC） | 回复 + 引用同上。 |
| Discord 残留 `.codex-lock` 文件 | 全新 —— 与 Iteration 2 race-fix 有交互 | 必须立刻改。 |
| Discord 静默 `flush()` 失败 | 与 Iteration 1 已锁定约束（"显式失败"）冲突 | 必须立刻改（约束违反）。 |
| #ISS-5104 `codex memory export` | **已在 Backlog（自 Iteration 2 起）**（Reviewer B 当时建议过） | 若 reviewer 愿意 PR，提升为 v0.6 epic。 |

### Step 2 —— 保留

- 新原话反馈追加到 `feedback.md`，置于 `## 2026-05-22 —— v0.5 上线后反馈`
  下。
- Iteration 2 的 `feedback.md`（已合并 Iteration 1）在做任何缩减之前，
  快照到 `archive/feedback-iter-2.md`。

### Step 3 —— 分类

| 类别 | 内容 |
|---|---|
| **必须立刻改（v0.5.1 patch）** | `memory list` 性能 regression（#ISS-5101）。pre-existing `~/.codex/memory/` panic（#ISS-5102）。残留 lockfile 清理。静默 `flush()` 失败（违反约束）。 |
| **保持稳定** | 存储后端（flat+lock，Iter 2 决定）。按项目隔离。opt-in flag 仍开。 |
| **意图** | 兑现原 RFC 的契约；patch 不能改变存储 shape。 |
| **证据** | #ISS-5102 的 stack trace；@u_ray 报告的 spinner 与文件状态不一致。 |
| **约束** | 热修补 patch —— 无 breaking change；v0.5.1 必须能 drop-in。 |
| **待决问题** | 本轮无阻塞。（multi-modal memory 与加密的 RFC 仍开放，但不在本轮路径上。） |
| **Backlog → v0.6 候选** | `codex memory export`（现在有志愿 PR 作者）。Multi-modal memory follow-up RFC。Encryption-at-rest follow-up RFC。 |

> **关键：multi-modal memory 与加密的请求在本轮 *不是新工作*。**
> 它们在 Iteration 1 的 `findings.md` 里已经 defer 过了，agent 的回复
> 显式引用现有条目，而不是重新挑起讨论。

### Step 4 —— `next_prompt.md`（为 v0.5.1 重新生成）

```markdown
# 下一轮 Prompt —— v0.5.1 patch release

## 上下文
v0.5 在 2026-05-15 上线了持久化 agent memory。一周用户反馈暴露 4 个 bug、
1 项约束违反、3 个早已 deferred 的点子。本轮发布 patch；deferred 的点子
继续 deferred。

## 必须立刻改（v0.5.1）
1. 排查并修 `codex memory list` 的性能 regression（#ISS-5101）。加一个
   benchmark 防止以后再 regress。
2. 修复升级后首次运行 `~/.codex/memory/` 已存在导致的 panic（#ISS-5102）。
   迁移路径：检测、warn、提供 `--migrate` 子命令。
3. 启动时清理残留 `.codex-lock`（当无 live PID 持有时）。
4. 修复 `flush()` 在 lock 超时时的静默失败。surface 为显式 error，符合
   已锁定的"显式失败"约束（findings.md 2026-04-22）。

## 保持稳定
- 存储后端（flat-file + advisory lock）。2026-05-08 决定，不再 revisit。
- 按项目隔离。
- v0.5 公共 CLI 表面（patch 内不能 breaking）。

## 承认但本轮 *不做*
- Multi-modal memory（#ISS-5103）→ **自 Iteration 1 起已在 Backlog。**
  回复链接到 `findings.md` 2026-04-22 和 deferred-RFC 计划。
- 静态加密（Discord @u_pat）→ **`findings.md` 中的 Q4，延后到 follow-up
  RFC。** 同样回复。
- `codex memory export`（#ISS-5104）→ **自 Iteration 2 起已在 Backlog。**
  若 @u_sam 愿意 PR，提升为 v0.6 epic；本轮不开始。

## 约束
- patch 发布；无 breaking change。
- "显式失败"规则（Iteration 1 锁定）适用于所有新 error 路径。

## 验收标准
- `cargo bench memory_list` 显示性能在 regression 前的 1.5× 之内。
- 手动复现 #ISS-5102（pre-existing 目录）：无 panic；清晰 warning；
  `--migrate` 工作。
- killed-process 复现下 `.codex-lock` 清理通过测试。
- `flush()` 在 lock 超时返回 `Err`，UI surface（不再静默成功）。
- 三项 "本轮不做" 都有一条 issue 回复，按日期引用现有 `findings.md` 条目。
```

### Step 5 —— 增量更新计划文件

`task_plan.md`（diff）：

```diff
 # task_plan.md

 ## Epic —— 持久化 Agent Memory

 ### In progress
 ...
+- [ ] v0.5.1 性能修复（#ISS-5101）
+- [ ] v0.5.1 pre-existing memory 目录迁移（#ISS-5102）
+- [ ] v0.5.1 残留 lockfile 清理
+- [ ] v0.5.1 flush lock 超时改为显式失败（约束修复）

 ### Backlog
 - [ ] Multi-modal memory                     （自 2026-04-22 deferred）
 - [ ] 超出按项目隔离的多租户保证               （自 2026-04-22 deferred）
-- [ ] `codex memory export` 子命令
+- [ ] `codex memory export` 子命令            （v0.6 候选；@u_sam 提议 PR）
+- [ ] 静态加密 follow-up RFC                  （自 2026-04-22 deferred，Q4）
```

`findings.md`（diff）：

```diff
 ## 2026-04-22 —— Discussion → RFC 阶段
 ...
 ## 2026-05-08 —— PR #PR-4123 review
 ...
+## 2026-05-22 —— v0.5 上线后
+
+### 重新确认（无变化）
+- 存储后端 = flat-file + advisory lock（2026-05-08 决定）。
+- 按项目隔离。
+- "显式失败"规则适用于所有新 error 路径 —— 据此驳回 @u_ray 报告的
+  静默 `flush()` 行为。
+
+### 新增约束（patch-release 范围）
+- v0.5.1 是 patch；不允许 breaking CLI 变更或存储 shape 变更。
+
+### 重新确认 deferral（无新争论）
+- Multi-modal memory：自 2026-04-22 起 deferred（Backlog）。
+- 静态加密：Q4，延后到 follow-up RFC。
+- `codex memory export`：自 2026-05-08 起 deferred（现为 v0.6 候选）。
```

`progress.md`：

```markdown
## 2026-05-22 —— Iteration 3
- v0.5 上线后 triage 完成。
- 4 项 patch 任务为 v0.5.1 unblock；3 项 deferred 已重新确认，
  附按日期引用 findings.md 条目的显式 issue 回复。
- archive/ 现含 feedback-iter-1.md 与 feedback-iter-2.md。
- 状态：v0.5.1 实施可开始；v0.6 epic 在 @u_sam 确认 PR 意向后再单独规划。
```

`changelog.md`：

```markdown
## 2026-05-22 —— Iteration 3: v0.5 上线后
- task_plan.md 加 4 项 v0.5.1 patch 任务。
- 3 项 deferred 重新确认；准备好显式 issue 回复。
- findings.md：重新确认 Iter 1 + Iter 2 决策；新增 v0.5.1 patch-scope 约束。
- 创建 archive/feedback-iter-2.md。
- 暂未修改源代码。
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
?? archive/feedback-iter-2.md

$ git log --oneline -- changelog.md
xyz9876 docs: iter 3 post-launch triage
abc1234 docs: post-PR-4123 plan update
def5678 docs: re-decide Q1 (flat+lock); race-fix landed
9876543 docs: RFC stage open decisions
```

## 4. 文件式 memory 实际兑现的价值

看本轮 agent **没有** 做的三件事：

1. **没有重新讨论 multi-modal memory。** 它在 `archive/feedback-iter-1.md`
   里找到了 `@u_henry` 的名字，匹配到 Iteration 1 的 Backlog 条目，
   于是用一条链接回复，而不是开一个并行的设计讨论。
2. **没有重新决定存储后端。** 即使有一条评论触及 lockfile 行为，agent
   识别出存储选择已经在 Iteration 2 最终决定，仅修复该选择内部的症状。
3. **没有静默地违反"显式失败"约束**，在处理 lock 超时报告时。它伸手到
   `findings.md` 2026-04-22，识别出违反，然后把修复列为 "必须立刻改"
   —— 即使没有任何 v0.5.1 用户按名字提到这条约束。

最后一条是最强的演示：Iteration 1 设下的一条约束，**在 Iteration 3 仍
积极塑造行为**，且无需任何人类重述。这就是文件式 memory 给你买来的东西。

⬅ 返回：[Iteration 2 —— PR review](iteration-2-pr-review.md)
⏪ 重新开始：[Iteration 1 —— RFC triage](iteration-1-rfc-triage.md)
