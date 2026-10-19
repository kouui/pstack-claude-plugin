# pstack 指南

当你不再对 agent 事无巨细地指手画脚时，pstack 的效果最好。你描述你想要什么，以及你将如何判断它已经完成。`/poteto-mode` 会选择 playbook，在步骤需要时调用其他 skill，并把证据展示给你。本指南用贴近实际的提示来讲解这种习惯。

你将学到以下内容：

1. [安装 pstack](./01-setup.md)。安装插件并选择你的模型。
2. [通过 `/poteto-mode` 分配工作](./02-poteto-mode.md)。给它一个目标，看它选择一个 playbook。
3. [理解代码](./03-understand.md)。先做只读调查，然后使用 `/how`、`/why`、`/teach` 和 `/recall`，之后再编辑任何东西。
4. [设计改动](./04-design.md)。在代码定型之前使用 `/architect`、`/arena`、`/swarm`、`/interrogate`、原型和计划。
5. [构建并清理改动](./05-build-and-clean.md)。构建类 playbook、`/tdd`、`/unslop` 和 `/no-comments`。
6. [验证并交付](./06-verify-and-ship.md)。在真实应用上证明行为，对测试很少覆盖到的部分做形式化验证，用 `/benchmark-checklist` 核查数字，然后开一个聚焦的 PR 并推进到合并。
7. [你睡觉时让工作继续运行](./07-overnight.md)。循环之前的信任、过夜约定、可供审计的决策日志，以及可扩展到多个 agent 的 Project 和自动化。
8. [用原则名称引导 agent](./08-principles.md)。在任务进行中用于重定向 agent 的 24 个名称。
9. [定制你自己的版本](./09-make-it-yours.md)。你自己的模式、针对重复错误的 `/correct`，以及如何测试 skill 的改动。
10. [配方与陷阱](./10-recipes-and-pitfalls.md)。可以直接复制的提示，以及应该避开的错误。

第一次阅读请按顺序阅读各页。之后，每一页都可以独立阅读。

当你卡住，或者不确定哪个 skill 适合时，请输入 [`/poteto-help`](../../skills/poteto-help/SKILL.md) 并附上你的问题：

```text
/poteto-help which skill should i use to review this branch?
```

它会给出回答，交给你一条可以发送的提示，并链接到得出该答案所依据的 skill 或指南页面。它不会启动工作，因为一次 pstack 运行会消耗真实的 token，所以你准备好时再发送提示即可。它只在你输入时才会运行。

## 如果你只记住一件事

用你自己的话，给 agent 一个目标和一种检查它的方式：

```text
/poteto-mode the export writes duplicate rows when a retry lands mid-run. repro first, then fix and verify.
```

你不需要指定 playbook，也不需要列出 skill。"repro first" 加上一个可检查的结果，就是 `/poteto-mode` 所需的全部路由信号。它会匹配 Bug fix playbook，把步骤复制进待办列表，并在每个步骤触发时调用正确的 skill。

下一步：[安装 pstack](./01-setup.md)。
