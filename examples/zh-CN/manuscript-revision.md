# 示例 3 —— 论文修改反馈

[English](../manuscript-revision.md) | [简体中文](manuscript-revision.md)

> 场景：研究者拿回了一份论文 v3 的长篇 review。其中既有具体修正、口味偏好，
> 也有 **两条相互冲突的建议**、一个开放的方法论问题，以及一个适合后续单独
> 写一篇论文的延展点子。智能体必须在不轻率消解冲突的前提下保留每一条评论。

---

## 1. 原始反馈（按原话保留）

> "Section 1 introduction 还行，但贡献点埋得太深；如果可能的话把三条贡献点
> 移到第一页。Section 2 related work 太长了，砍掉 30%。Section 3 method ——
> 我觉得应该重新表述为对 Method X 的扩展，而不是一个新框架。
> 但 Reviewer 2 明确说不要把它写成扩展，因为 loss 公式确实不一样 —— 你来
> 协调一下这个分歧。
>
> Section 4 experiments —— Table 3 不可读，重排版。加一列置信区间。
>
> Section 5 discussion —— 推测过多，要么收紧，要么把推测部分单独放到一个
> 'limitations and future work' 子节里。
>
> 问题：重投之前要不要在更大数据集上跑一次 ablation？我倾向跑，但要花
> ~2 天 compute。
>
> 引用：在 Section 3 加上 [Smith 2024] 和 [Liu 2024]。
>
> 后续 paper 的小点子：这个工作的 multi-modal 版本。这一轮不要做，留着以后。"

## 2. skill 一步步做了什么

### Step 1 —— 定位

```
$ git status --short
 M paper/main.tex
 M paper/sections/method.tex
?? notes/feedback-2026-04-18.md
```

读 `task_plan.md`（论文的 milestone）和 `findings.md`（关于 framing 的历史决策）。

### Step 2 —— 保留

原话 review 进入 `feedback.md`。之前更短的 `feedback.md` 先快照到
`archive/feedback-2026-04-09.md`。

### Step 3 —— 分类

| 类别 | 内容 |
|---|---|
| **必须立刻改** | 三条贡献点移到第一页。Section 2 砍 30%。Table 3 重排版。Table 3 加 CI 列。Section 5 收紧（或拆出 "limitations and future work" 子节）。Section 3 加引用 [Smith 2024]、[Liu 2024]。 |
| **保持稳定** | （隐含）Section 1 intro 和 Section 4 experiments 叙述本身 —— 只是 Table 3 要重排，实验本身不动。 |
| **意图** | 让贡献点早一点显眼；砍掉非必要叙述；论断要 defensible。 |
| **证据** | Method framing 的两条评论冲突（导师 vs Reviewer 2）。 |
| **约束** | Compute 预算：更大数据集 ablation 要 ~2 天。 |
| **待澄清问题** | (a) Method framing —— 写成 Method X 的扩展，还是新框架？(b) 重投之前要不要跑更大数据集的 ablation？ |
| **待办池** | 多模态 follow-up paper。 |

> **关键：冲突被保留，而不是被消解。** skill 不会偷偷在 framing 上选边站。
> 它会把两边的 review 都摆到 `next_prompt.md` 的 **未决问题** 一节里，
> 附上各自的证据。

## 3. 产出的 `next_prompt.md`

```markdown
# 下一轮 Prompt —— 论文修改（round 2）

## 上下文
论文 v3 在 2026-04-18 收到导师 + Reviewer 2 评论。
目标：~2 周内重投。Compute 预算：约 2 天空闲。

## 必须立刻改
1. 把三条贡献点移到第 1 页（intro 部分）。
2. Section 2（Related Work）砍 ≈30%。保留 [Doe 2023] 和 [Wang 2024] 的覆盖；
   砍历史背景叙述。
3. Section 4 的 Table 3 重排版以提升可读性：
   - 单列对应单方法的布局。
   - 加 95% 置信区间列。
   - 每行最佳结果加粗。
4. 收紧 Section 5（Discussion）：所有推测性论断挪到新建的 "Limitations and
   Future Work" 子节。正文只保留有直接证据支持的讨论。
5. Section 3 中加引用：
   - [Smith 2024] 在 loss 公式那段附近。
   - [Liu 2024] 在相关理论小节附近。

## 保持稳定
- Section 1 intro 叙述本身（只挪贡献点）。
- Section 4 experiments 的叙述和主结果论断。
- 已经定义过的符号和记号。

## 待澄清问题（决定后再继续重写对应小节）

### Q1 —— Method framing（导师 vs Reviewer 2 冲突）
- **导师** 建议：把 Section 3 重新表述为 Method X 的扩展。
- **Reviewer 2** 明确说：不要写成扩展，因为 loss 公式确实不一样。
- 待权衡的证据：
  - loss 中确实有一个 Method X 没有的新项。
  - 优化过程是相同的。
  - 在 5 个 benchmark 中有 4 个胜过 Method X。
- 可能的解决方案：
  (a) 写成扩展；把新 loss 项作为贡献点之一。
  (b) 写成新框架；明确把 Method X 列为最近的 prior。
  (c) 混合写法："我们建立在 Method X 之上（Section 3.1），但引入一个新 loss
      （Section 3.2），它在公式上带来实质变化。"
- **重写 Section 3 之前必须决定。** 推荐：(c)。

### Q2 —— 更大数据集 ablation
- Compute 成本：约 2 天。
- 不跑的风险：reviewer 可能挑战泛化能力。
- 跑的风险：重投延迟 2 天。
- 如预算允许：建议跑。

## 待办池（本轮不做）
- 多模态扩展论文。

## 约束
- 除 Q2 中的更大数据集 ablation 外，本轮不做新实验。
- 保持已经使用的 LaTeX label 和 bibkey 命名规范。

## 验收标准
- 第 1 页含 3 条带编号的 "Contributions" 列表。
- `wc -w paper/sections/related-work.tex` 显示相对上一个 commit 减少 ~30%。
- Table 3 在单栏宽度下不会横向溢出。
- [Smith 2024] 和 [Liu 2024] 都出现在 `paper/refs.bib` 中且在 Section 3 被引用。
- Section 5 含一个清晰分隔的 "Limitations and Future Work" 子节。
- 共同作者能在 10 分钟内 review 完整个 `git diff`。
```

## 4. 更新计划文件（仅增量）

### `task_plan.md`（diff 视图）

```diff
 ## 论文修改
-- [ ] 处理 round-1 reviewer 反馈
+- [x] 处理 round-1 reviewer 反馈
+- [ ] 把贡献点移到第一页
+- [ ] Related work 砍 ~30%
+- [ ] Table 3 重排 + 加 CI 列
+- [ ] Section 5 收紧 / 拆出 limitations 子节
+- [ ] 加引用 [Smith 2024]、[Liu 2024]
+- [ ] （待决策）Method framing —— 见 findings.md Q1
+- [ ] （待决策）跑更大数据集的 ablation —— 见 findings.md Q2

 ## Backlog
+- [ ] 多模态扩展论文
```

### `findings.md`（节选）

```markdown
## 2026-04-18 —— 未决方法论问题

### Q1 —— Method framing
导师与 Reviewer 2 之间的冲突。两边观点已记录在 next_prompt.md。
**尚未决定。** 一旦决定，把选择锁在这里，避免后续重写时反复争论。

### Q2 —— 更大数据集 ablation
未决。在开始 Section 4 重写之前应先决定 —— 因为新加的 CI 列要基于哪一份数据集
的结果。
```

### `progress.md`

```markdown
## 2026-04-18

- 收到 round-2 review（导师 + Reviewer 2）。
- 5 项必改已 unblocked；2 项问题阻塞；1 项 backlog。
- 暂未修改论文正文。
- 下一步阻塞决策：framing（Q1）、ablation 范围（Q2）。
```

### `changelog.md`

```markdown
## 2026-04-18

- review 论文 v3；处理导师 + Reviewer 2 评论。
- 更新 task_plan.md（+5 任务，+2 决策项，+1 backlog）。
- 在 findings.md 登记未决方法论问题。
- 生成 next_prompt.md，将冲突评论显式摆出。
- archive/feedback-2026-04-09.md 已从原 feedback.md 创建。
```

## 5. Git 复审

```
$ git status --short
 M task_plan.md
 M findings.md
 M progress.md
 M changelog.md
?? feedback.md
?? next_prompt.md
?? archive/feedback-2026-04-09.md
```

智能体回复用户时会提到：

- 6 个文件已更新。
- 2 项未决决策（在 findings.md 中记为 Q1 和 Q2）。
- 暂未修改论文正文；在 Q1 和 Q2 决定之前不会动 Section 3 和 Section 4。
- 建议下一条消息：*"Q1 用方案 (c)，Q2 跑 ablation。"* —— 收到后智能体就能
  把 next_prompt.md 转化为真正的 LaTeX 编辑。

---

> **要点** —— 篇幅长、语气混杂的学术 review（包含一处 reviewer 与导师之间的
> 冲突）被转化为一份清晰的增量计划改动、两个 framing 良好的未决决策，
> 以及一条审计链。没有过早消解冲突；没有丢失任何源材料。
