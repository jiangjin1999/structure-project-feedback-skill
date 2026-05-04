# 实战示例

[English](../README.md) | [简体中文](README.md)

`structure-project-feedback` 在真实场景中的端到端走查。
每个示例展示：

1. 用户刚刚 review 的原始产物。
2. 用户丢给智能体的 **原始反馈**（按原话保留）。
3. skill 是如何 **分类** 的。
4. skill 产出/更新的每一个文件（`next_prompt.md`、
   `task_plan.md`、`progress.md`、`changelog.md`、`archive/…`）。
5. reviewer 在 Git 中能看到的 diff。

| # | 场景 | 什么时候来读这个 |
|---|---|---|
| 1 | [PPT 反馈](slide-deck-feedback.md) | PPT 草稿 review 后混着风格 + 内容 + 待办的反馈。 |
| 2 | [代码 PR review 反馈](code-review-feedback.md) | 多条 reviewer 评论 —— 修正 + nit + 大改建议混合。 |
| 3 | [论文 / 长文修改反馈](manuscript-revision.md) | 学术 review，含冲突评论和未决问题。 |

> 所有示例都刻意保持通用性。**不来自任何具体的私有项目** —— 你可以自由复制
> 和改编到自己的 AGENTS.md 中。
