---
name: how
description: "用于 \"how does X work\"、在改动前的代码走读、以及位置/归属/分层类问题（\"where should this live\"、\"which package owns this\"、\"is this the right layer\"）。解释子系统架构、运行时流程和入门心智模型。动机类问题请使用 why。"
disable-model-invocation: true
---

# How

在 Claude Code 中，遵循本技能之前，每个会话先阅读一次[Claude Code 适配器](../../docs/harness/claude-code.md)。它将 pstack 技能中的 Cursor 名称映射到 Claude Code。

探索代码库以回答 "how does X work?" 类问题。输出架构说明，水平相当于为子系统入门的资深工程师，足以建立可用的心智模型，但不要详细到像带注释的源代码。

每次生成都是一个 Task 子代理，使用 `subagent_type: generalPurpose` 和 `readonly: true`。探索者使用 `pstack-models.md` 规则中的 `how explorer` 行（默认 `sonnet high`），每个解释者使用 `how explainer` 行（默认 `opus high`）。将 `model` 设置为该行的值；如果规则或该行缺失，则设置为默认值。当值为 `auto` 或 `inherit-parent` 时，保持 `model` 未设置。如果 Task 工具拒绝某个 slug，则使用默认值并说明这一点。如果它也拒绝默认值，则使用其错误信息中同一系列中最接近的有效 slug。按照 **poteto-mode** 技能的子代理部分，为该工作选择每个子代理的推理级别以及快速或标准速度。

## 1. 评估复杂度

如果范围不明确，说明你的理解并进行探索。用户可以纠正方向。

- **简单**（单个模块、小型工具、狭窄的问题，例如 "how does function X work"）：不使用探索者。生成一个解释者，它在一次过程中完成探索和解释，使用 `references/explainer-prompt.md` 构建，且不包含探索者发现部分。进入第 4 步。
- **复杂**（跨多个文件或服务的子系统、横切功能、完整的架构概览）：进入第 2 步。

存在疑问时，走简单路径。

## 2. 探索（仅复杂情况）

将问题分解为 2 到 4 个角度，每个角度是子系统的一个不同切片。在同一条消息中生成所有探索者，每个探索者都使用 `references/explorer-prompt.md` 并填入其角度。

## 3. 综合（仅复杂情况）

所有探索者返回后，生成一个解释者，使用 `references/explainer-prompt.md` 并填入每个探索者的发现。

## 4. 呈现

呈现解释者的输出。为清晰度或结合对话上下文进行的轻微编辑是可以的。不要大幅改写。

## 输出格式

解释使用 `references/explainer-prompt.md` 中定义的章节，省略不适用的章节：Overview、Key Concepts、How It Works、Where Things Live、Gotchas。
