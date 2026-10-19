---
name: no-comments
description: "派出 Comment Sicko，修复已接受的发现，并为声明的约束提供编码方案。"
disable-model-invocation: true
---

# No comments

在 Claude Code 中，遵循本 skill 之前，每个会话先阅读一次 [the Claude Code adapter](../../docs/harness/claude-code.md)。它把 pstack skills 中的 Cursor 名称映射到 Claude Code。

派出 Comment Sicko。对已接受的发现采取行动。

尊重 Comment Sicko 的全新视角。

## 范围

使用调用方指定的文件或 diff。否则使用当前分支相对于基线分支（默认 `main`）的 diff，包括工作区中的改动。

## 步骤

1. 使用 `subagent_type: "comment-sicko"` 派生 `Task`。传入范围。根据任务为每个 subagent 选择推理级别以及 fast 或 standard 速度，依据 **poteto-mode** skill 的 Subagents 部分。不要重述其规则。
2. 检查其报告和 diff。拒绝对应用代码的编辑、超出范围的操作、受例外保护的删除、对 `MUST KILL` 理由的错误陈述，以及把保留的有意代码判为有罪的标记。针对我方代码的意外情况所给出的重塑标记仍然必须可执行。不要恢复这些注释。只有在有证据表明保留的内容涉及我们无法更改的事物时，保留才能成立。审计遗漏的、限定范围的 lint 和 TypeScript 抑制。涉及正确性或安全性的抑制仍然是可执行的 `MUST KILL`。只有附带精确例外和限定范围证据时，才恢复删除。在接受薄弱的 `IMPORTANT` 或 `do not remove` 删除或保留之前，对其符号运行 `/how` 或 `/why`。如果某个删除判断含糊，则不恢复。如果某个保留被驳倒或仍然含糊，则删除它。撤销并针对被拒绝的报告重新运行一次，并写明失败原因。若第二次仍被拒绝，则将其报告为未结项，并使 `/no-comments` 失败。
3. 通过删除死路径、去掉参数或使用真实 API，直接修复已接受的琐碎标记。如果任何修复需要确定形态，则对已接受的整组发现及其周边代码运行一次 `/architect`。停在草图处。由 Architect 确定形态。第 4 步负责实现。
4. 在范围内实现最小的根因修复。移除每一个被点名的变通方案。如果根因超出范围，则落地范围内最小的修复，并将其余部分报告为未结项。**principle-fix-root-causes** 和 **principle-redesign-from-first-principles** skill 仅指导意图。两者都不授权扩大范围边界，也不授权修复边界之外的实例。绝不叠加症状性防护。
5. 约束类注释写明 `do not remove`、`do not change wording` 或 `talk to X before changing`。对于我们无法更改的事物，保留这些注释。提供范围内成本最低的类型、运行时、测试或 CI lint。等待交互式批准。无人值守和评估场景需要调用方预先批准。如果获得批准，则先编码再删除。否则直接删除，将该约束报告为未结项，并勾勒范围外的工作草图。
6. 报告删除数量、恢复的注释、重新运行次数、architect 草图、修复、编码方案提议、编码结果、未被强制执行的约束，以及其他未结工作。
