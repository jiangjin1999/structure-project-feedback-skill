# 示例 1 —— PPT 反馈

[English](../slide-deck-feedback.md) | [简体中文](slide-deck-feedback.md)

> 场景：一位设计师刚把 12 页 PPT 草稿发给同事 review。同事甩回一段
> 混合反馈（修正 + 口味 + 跑题点子 + 一个待澄清的问题 + 一句"这块别动"）。
> 智能体已经装了 `structure-project-feedback` skill。

---

## 1. 原始反馈（按原话保留）

> "前两页感觉对的。第 3 页太挤了 —— bullet 太多。可以试个不一样的配色吗？
> 想要不那么 corporate 的感觉。第 4 页我们到底想说什么来着？我忘了之前定下来的版本。
> 字体不要动。
> 案例研究的点子挺有意思，但这一轮先放着吧。
> 还有第 7 页右上角那个图标坏了。
> 哦对了，这份 deck 周五要给 leadership。"

## 2. skill 一步步做了什么

### Step 1 —— 定位

```
$ git status --short
 M slides/main.key
?? notes.txt
```

智能体读取已有计划文件：`task_plan.md`、`progress.md`、`changelog.md`，
然后看了眼 PPT 本身。

### Step 2 —— 保留

原始那段反馈被原话追加到 `feedback.md`，加上日期标题，**先于** 任何归纳：

```markdown
## 2026-04-12 —— v3 草稿 review

> 前两页感觉对的。第 3 页太挤了 —— bullet 太多。
> ……（按原话）……
```

之前的 `feedback.md`（如果做过缩减）会先快照到 `archive/feedback-2026-04-09.md`。

### Step 3 —— 分类

| 类别 | 内容 |
|---|---|
| **必须立刻改** | 降低第 3 页密度。尝试新配色。修第 7 页右上角坏图标。 |
| **保持稳定** | 第 1–2 页（已认可）。现有字体方案。 |
| **意图** | 整体感觉不要那么 corporate。 |
| **约束** | 硬性截止：周五（已对 leadership 承诺）。 |
| **待澄清问题** | 第 4 页核心信息 —— 之前定下来的是什么？改版前需要确认。 |
| **待办池** | 案例研究章节。 |

## 3. 产出的 `next_prompt.md`

```markdown
# 下一轮 Prompt —— Deck v4

## 上下文
v3 草稿 12 页，2026-04-12 review。目标：周五给 leadership。

## 必须立刻改
- 第 3 页：降到 ≤ 4 条 bullet，每条 ≤ 25 字。
- 配色：先给出 2–3 个备选方案（不那么 corporate；偏暖色 / 柔和色调可考虑），再选一个实施。
- 第 7 页：替换右上角坏图标。顺便检查所有图标是否正常渲染。

## 保持稳定
- 第 1–2 页（保留原样）。
- v3 字体方案（字号、字重、层级都不要动）。

## 意图
- 整体氛围更亲和、更现代，少一点 corporate 感。

## 约束
- 周五 EOD 硬截止（leadership review）。
- 配色备选方案需保证正文 WCAG AA 对比度。

## 待澄清问题
- 第 4 页核心信息 —— 改版前需要确认。从 notes.txt 看到两个候选：
  (a) "we ship faster than we plan" 或 (b) "we plan in writing, ship in steps"。

## 待办池（本轮不做）
- 案例研究章节。

## 验收标准
- 第 3 页字数 ≤ 100。
- 配色变体保存为 `theme-A.key` 和 `theme-B.key`。
- 所有图标正常加载（视觉走查 + 导出 PDF 校验）。
- 周五交付文件名：`deck-v4-final.key`。
```

## 4. 更新计划文件（仅增量）

### `task_plan.md`（diff 视图）

```diff
 ## In Progress
-- [ ] 完成草稿 deck v3
+- [x] 完成草稿 deck v3（2026-04-12 review）
+- [ ] 降低第 3 页密度
+- [ ] 产出 2–3 个配色变体
+- [ ] 替换第 7 页坏图标

 ## Open Questions
+- [ ] 改版前确认第 4 页核心信息

 ## Backlog
+- [ ] 案例研究章节（v4 推迟）
```

### `progress.md`（节选）

```markdown
## 2026-04-12

- v3 草稿交付 review；返回 6 项可执行待办。
- 状态：第 1–2 页通过；第 3、7 页待修；配色待探索；第 4 页待澄清。
- 下一里程碑：deck-v4，周五 EOD（硬截止）。
```

### `changelog.md`

```markdown
## 2026-04-12

- review 了 deck v3。
- 更新 `task_plan.md`：4 项新任务；案例研究移至 backlog。
- 生成 v4 的 `next_prompt.md`。
- 暂未修改产物本身。
```

## 5. Git 复审

```
$ git status --short
 M task_plan.md
 M progress.md
 M changelog.md
?? feedback.md
?? next_prompt.md

$ git diff --stat
 changelog.md   |  6 ++++++
 progress.md    |  5 +++++
 task_plan.md   |  9 +++++++--
 3 files changed, 18 insertions(+), 2 deletions(-)
```

智能体在回复中报告：变更的文件、未决问题、并等待用户：(a) 回答第 4 页的问题，
或 (b) 让它开始实施必须立刻改的项，或 (c) 直接 commit 计划更新即可。

---

> **要点** —— 用户那段含 7 个想法的混合反馈现在变成了：
> 一份原话存档、一张分类表、一份带验收标准的可执行 prompt、以及对项目计划文件的
> 增量更新 —— 全部都能在一个 Git diff 里 review 完。
