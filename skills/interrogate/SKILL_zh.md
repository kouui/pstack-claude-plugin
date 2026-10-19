---
name: interrogate
description: "用于 \"interrogate\"、\"adversarial review\"、\"multi-model review\"、\"challenge this\"、\"stress test this code\"、\"find blind spots\" 或 \"tear this apart\" 等请求。多个 LLM 评审者从相互独立的角度质疑变更。"
disable-model-invocation: true
---

# Interrogate

在 Claude Code 中，每个会话开始时先阅读一次 [Claude Code 适配器](../../docs/harness/claude-code.md)，然后再遵循本 skill。它把 pstack skills 中的 Cursor 名称映射到 Claude Code。

为每个已配置的模型派生一个评审者，对代码变更进行对抗式评审。对抗信号来自模型多样性，而不是指定的角色设定。交付物是综合结论。不要自动应用任何变更。

## 步骤 1，确定范围

根据上下文确定要评审的内容：

- 如果用户指定了具体文件或 diff，就使用它们。
- 如果位于功能分支上，运行 `git diff main...HEAD`（或相应的基础分支）以获取完整变更集。
- 如果用户的消息提到了近期工作，收集相关文件。

将 diff 或文件内容连同评审者理解代码所需的周边上下文文件一起打包。

## 步骤 2，意图

在派生评审者之前，根据用户的消息、commit 信息、已有的 PR 描述以及代码，用一段清晰的文字陈述意图。如果你不确定意图，在继续之前先询问用户。

## 步骤 3，派生评审者

在一条消息中使用 Task 工具同时启动所有评审者，每个评审者对应 `~/.claude/rules/pstack-models.md` 中 `interrogate reviewers` 行的一个条目，按条目数量依次标记为 Reviewer A、B 等。如果该规则或该行缺失，则使用表格中的默认值。

| 子代理 | 默认模型 |
|--------|----------|
| Reviewer A | `opus high` |
| Reviewer B | `sonnet high` |

每个评审者都要设置 `subagent_type: generalPurpose`、`readonly: true`，并将 `model` 设为其配置条目；如果没有配置该行，则设为表格中的默认值。对于 `auto` 或 `inherit-parent` 条目，省略 `model`，使该评审者运行在父模型上。

如果 Agent 工具拒绝某个配置条目，则让该评审者改用其表格默认值，并说明这一点。如果它拒绝某个表格默认值，则从错误信息中选择最接近的有效模型来派生（优先选择相同模型和推理档位），并另开一个 PR 来更新默认值表格。不要因为模型问题而阻塞评审。永远不要把别名条目当作被拒绝的模型，也不要对其应用回退逻辑。

阅读 `references/reviewer-prompt.md`，并为每个评审者填写同一模板，内容包括已声明的意图、diff 或文件内容、来自 `references/rubric.md` 的评分标准，以及来自 `references/code-quality-review.md` 的代码质量视角。按照 **poteto-mode** skill 中 Subagents 一节的规定，为每项任务选择各子代理的推理级别以及快速或标准速度。

## 步骤 4，综合

结果陆续返回时，建立统一的全局视图。解析每位评审者的全部发现。合并那些以不同方式描述同一问题的发现，并记录提出该发现的模型。由两个或更多模型独立提出的发现信号最强。单个模型提出的发现要阅读，但相应降低其权重。记录分歧。如果一个模型标记了某事项，而另一个模型明确表示相反的意见，这就是结论的上下文。

## 步骤 5，主审判断

你是主评审者，是务实的资深工程师，而不是中立的汇总者。完整框架请阅读 `references/lead-judgment.md`。将每项发现归入以下四类之一，并注明提出该发现的模型以及一行理由。

- **Act on（应处理）**。确实影响正确性、安全性或可维护性（就实际目标而言）的问题。它们会阻塞一个真实的 PR。
- **Consider（需考虑）**。站得住脚，但你不确定其价值是否超过现在处理的成本。值得用户关注。
- **Noted（备注）**。技术上成立，但不可执行。依赖上下文、过早优化，或在当前阶段影响较小。
- **Dismissed（已驳回）**。错误、吹毛求疵，或缺少上下文，附简要理由。

## 输出格式

### Intent（意图）
> [步骤 2 中陈述的意图段落]

### Reviewers（评审者）
- Reviewer [label]：[模型名称]，[N 项发现]（每位评审者一个条目）

### Act On（应处理）
[每项包括：描述、提出该问题的模型、为何重要。]

### Consider（需考虑）
[每项包括：描述、提出该问题的模型、权衡取舍。]

### Noted（备注）
[简要列表。]

### Dismissed（已驳回）
[每项附简要理由。]

### Agreement Map（一致性图）
[模型之间一致与分歧之处，以及这一模式说明了什么。]
