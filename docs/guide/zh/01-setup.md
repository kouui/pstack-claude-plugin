# 设置 pstack

本页说明如何安装插件、选择 pstack 使用的模型，并运行你的第一个任务。设置只需一条命令和一段简短的对话。

## 安装插件

在 Claude Code 中运行 `claude plugin install pstack@magic-claude-code-plugin -s user`。Claude Code 会为插件命令加上前缀，因此本指南中的每个命令在那里都要加 `pstack:` 前缀，例如 `/pstack:setup-pstack`，它会把模型规则写入 `~/.claude/rules/pstack-models.md`。[Claude Code 适配器](../../harness/claude-code.md)介绍了其他差异。

## 选择模型

运行：

```text
/setup-pstack
```

[`/setup-pstack`](../../../skills/setup-pstack/SKILL.md) 会检测你可以使用的模型，询问推理预算，展示每个角色（代码委派、判断、审查面板），然后询问你的需求。请回答这些问题。它会写入 `~/.claude/rules/pstack-models.md`，这是一条小规则，每个 pstack 技能都会读取它。

默认设置以 `high` 推理运行，即 `high` 预算。`medium` 预算会把每个角色的推理降为 `medium`，消耗更少的 token。本插件只使用这两个级别。

你只需覆盖你关心的角色。规则中没有某个角色的行时，该角色使用技能的默认值。要恢复默认值，删除该角色的那一行。重新运行 `/setup-pstack` 时，会保留模型与默认值不同的角色。默认值变更后，在变更之前写入的规则仍会固定旧的默认值，因此请删除这些角色行，或删除该文件，然后重新运行 `/setup-pstack`。

你可能想知道如果使用 Auto 会怎样。将某个角色设为 `inherit-parent` 或 `auto` 后，pstack 会省略子代理的 `model` 字段，子代理便继承你父级对话的模型。这两个值含义相同，且都不是模型标识。对于面板角色，该值是一个列表，每个条目运行一个子代理，因此列表长度决定面板大小。设置还会配置 `swarm workers`，即每个 `/swarm` 工作者的默认模型，除非某场竞赛为每个分支指定了模型。

## 接受验证提议，或者拒绝

设置结束时，`/setup-pstack` 会查找在你的项目中证明应用行为的方法，即 `verify-*` 技能或现有的测试工具（harness）。如果两者都找不到，它会提供一次机会，用 [`/create-verification-skill`](../../../skills/create-verification-skill/SKILL.md) 生成一个。

你同意后，它会写入 `.claude/skills/verify-<app>/`，这是一个项目本地技能，教代理像用户一样操作你的应用。交付前它会先证明该技能可以正常工作。你拒绝的话，设置会直接继续。你随时可以自己运行 `/create-verification-skill`。[验证与交付](./06-verify-and-ship.md#create-a-project-verification-skill)对此有深入介绍。

如果你是 pstack 新手，请同意。能够检查自己工作的代理会一直做下去，直到检查通过；不能检查的代理则会把每个结果交回给你，由你手动核对。在本指南的所有内容中，验证技能带来的收益最大。

然后设置会提供一次机会，用 [`/create-formal-verification-skill`](../../../skills/create-formal-verification-skill/SKILL.md) 生成正式的验证技能，除非项目已经有了。测试只检查有人想到并写下的用例。正式验证技能则在既定范围内的所有情况下，检查你代码的一个关键属性，例如"一个任务永远不会同时运行两次"。如果你的代码涉及并发、重试、状态机或解析器，且一个缺陷代价高昂、测试很难覆盖到会触发它的情况，就请同意。否则请拒绝，情况变化后随时可以再运行。[验证与交付](./06-verify-and-ship.md#prove-a-property-with-a-formal-verification-skill)对此有说明。

设置完成后，请开启新的对话。模型规则只对新会话生效。

## 控制成本

pstack 会在子代理和审查面板上额外消耗 token，这是严谨性的代价。若想少花一些：

- 重新运行 `/setup-pstack`，选择更小的推理预算或更便宜的模型。在主对话中使用强模型，在代码角色中使用更便宜、更快的模型，是一种很好的搭配。
- 将某个角色设为 `auto` 或 `inherit-parent`，使其使用对话自身的模型。
- 缩短面板列表。每个条目都会运行一个子代理。
- 把 `/poteto-mode` 留给需要严谨性的工作。小而明显的修改不需要它。

## 运行你的第一个任务

选一件真实但较小的事情，像向同事描述那样描述它：

```text
/poteto-mode add a --json flag to this command. text output stays byte-identical. verify both.
```

观察待办列表。它的前几项是匹配到的操作手册步骤，即针对此提示的 Feature 手册。如果 `/poteto-mode` 跳过了某个步骤，该步骤会保留在列表中并标注 `skip: <reason>`，这样你就能看到它选择不做什么。

下一步：[通过 `/poteto-mode` 分派工作](./02-poteto-mode.md)。
