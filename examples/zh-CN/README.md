# 实战示例 —— 一个贯穿三轮的迭代故事

[English](../README.md) | [简体中文](README.md)

这些示例构成 **同一个连贯故事**：在一个真实知名开源项目上，跨三轮反馈维护
一个功能。它们专门用来展示 skill 中最难用单一示例说清楚的那部分价值 ——
**`task_plan.md`、`findings.md`、`progress.md`、`changelog.md` 在多轮迭代之间
作为项目的"长期文件式 memory"是如何累积的**。

## 案例项目

我们用 [**openai/codex**](https://github.com/openai/codex) 作为案例项目 —— OpenAI
开源的命令行 coding agent（约 80k stars，Apache-2.0）。选它有几个原因：

- Codex 本身就遵循 SKILL.md / AGENTS.md 规范，所以我们是在一个原生理解
  这套规范的项目上演示这个 skill。
- 它有真实社区、真实 issue、真实"多方意见混合"的功能决策 —— 正是这个
  skill 想吸收的那种语境。
- Apache-2.0、完全公开 —— 任何人都可以 fork 然后复现这套循环。

> 关于 PR / issue 编号：我们使用示意性 ID（`#PR-4123`、`#ISS-4099` 等），
> 这样示例足够具体，又不会与任何具体的真实 PR 混淆。被设计的功能本身
> —— "跨 session 的 agent 持久化 memory" —— 是 codex 社区中真实反复出现的话题。

## 故事

我们追踪同一个功能跨三轮迭代：

> **目标：** 给 `codex` 加上跨 session 的持久化 agent memory，让 AGENTS.md /
> skill 状态、findings、approval 决策能在多次运行间保留下来。

| 迭代 | 触发点 | skill 的角色 | 关键文件更新 |
|---|---|---|---|
| [**1 —— RFC triage**](iteration-1-rfc-triage.md) | 一条含 12 条评论的 GitHub Discussion，混着提议、冲突和跑题 | 把讨论转化为可执行的 RFC 计划，记录未决决策 | `feedback.md`、`next_prompt.md`、`task_plan.md`、`findings.md`、`progress.md`、`changelog.md` |
| [**2 —— PR review**](iteration-2-pr-review.md) | 实施 PR 收到 reviewer 混合评论，其中一位 **挑战了 Iteration 1 的决策** | 在处理评论前先读 `findings.md`，对被挑战的决策要么确认、要么重新打开为未决问题 | 同样几个文件，**仅做增量更新**；新增 `archive/feedback-iter-1.md` |
| [**3 —— 上线后用户反馈**](iteration-3-post-launch-feedback.md) | v0.5 上线后，用户提了一堆混合反馈：bug、新点子，以及一条 **Iteration 1 已经 deferred 的提议** | 把新反馈与现有 `findings.md` 和 `Backlog` 交叉比对，避免重复争论 | 同样几个文件；`archive/` 现在有 2 份快照；`changelog.md` 已能展示完整的 v0.4 → v0.5 → v0.5.1 链 |

## 这个故事想展示的是什么

一个 skill 真正有用，是因为它让 **下一轮** 比上一轮更省力。读完这三个示例
（按顺序）之后，你应该能回答：

1. *Iteration 2 的 agent 知道 Iteration 1 做出了哪些决策？这些知识存在哪里？*
   → 在 `findings.md`、以及 `archive/` 里前一轮 `feedback.md` 的快照里。
2. *是什么阻止了 Iteration 3 重复讨论一个早就决定过的问题？*
   → Iteration 1 在 `findings.md` 里登记的条目，加上 Iteration 1 prompt 里
   显式列出的 `Backlog` 一节。
3. *换一个 agent —— 或一个新人 —— 能不能从 Iteration 3 接手项目，无需任何
   先验上下文？*
   → 能，靠读 `task_plan.md`、`findings.md`、`progress.md`、`changelog.md`
   和最新的 `next_prompt.md`。

这个"往返链路"才是这个 skill 真正约束住的契约。

## 阅读顺序

1. [Iteration 1 —— RFC triage（来自 GitHub Discussion）](iteration-1-rfc-triage.md)
2. [Iteration 2 —— PR review（含对前一轮决策的挑战）](iteration-2-pr-review.md)
3. [Iteration 3 —— 上线后用户反馈（含 deferred idea 的复活）](iteration-3-post-launch-feedback.md)

> 所有反馈内容、文件状态、PR/issue ID 都是示意性的。请按你自己的仓库和
> 团队工具习惯改编。
