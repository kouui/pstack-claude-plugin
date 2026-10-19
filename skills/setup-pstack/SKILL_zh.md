---
name: setup-pstack
description: 配置 pstack 在每个角色上使用哪些模型以及使用多大的推理预算。检测可用模型,并写入一条始终生效的规则,覆盖 skill 默认值。用于 /setup-pstack、"configure pstack models"、"pstack budget",或更改 pstack 的模型选择。
---

# Setup pstack

在 Claude Code 中,每个会话开始时先阅读一次 [Claude Code 适配器](../../docs/harness/claude-code.md),然后再按本 skill 操作。它把 pstack skill 中的 Cursor 名称映射到 Claude Code。

编写 `~/.claude/rules/pstack-models.md`,这是一条按角色设置 pstack 模型的规则。Claude Code 会把 `~/.claude/rules/` 中的每个 `.md` 文件加载到每个新会话中。

本插件目前支持三种模型:`opus`、`sonnet`、`haiku`(5.5 系列)。以后可以添加其他模型。

## 步骤

### 1. 检测可用模型

读取 `Agent` 工具 schema 中列出的 `model` 值,只保留 `opus`、`sonnet` 和 `haiku`。如果无法读取,请询问用户这三个中哪些可用。绝不写入你没有确认可用的模型。别名 `inherit-parent` 和 `auto` 始终有效,尽管它们不是模型。

### 2. 加载当前状态

默认的角色到模型映射是下方第 5 步中展示的规则格式。如果 `~/.claude/rules/pstack-models.md` 已存在,请读取它,并把其中的 `# budget` 行和各角色的值视为当前选择。否则从这些默认值开始。角色不在第 5 步中的行(例如 `how critics`)来自已退役的角色,请丢弃。

### 3. 预算、映射与确认

**(a) 询问预算。** 优先使用 AskUserQuestion,而不是自由文本;在 `-p` 模式下以纯文本询问。提供以下两个选项,标签必须完全一致。当规则中记录了当前预算时,说明当前预算是什么。没有规则时,说明 `high` 与 skill 默认值一致。

- `high — high reasoning`
- `medium — medium reasoning`

**(b) 应用预算。** 以 skill 默认值构建工作表;重新运行时,保留你已更改模型的角色。角色值的格式为 `<model> <effort>`,effort 只能是 `high` 或 `medium`。预算设置每个真实条目的 effort,包括 panel 条目。`inherit-parent` 和 `auto` 不变。因此 `medium` 会把 `sonnet high` 变为 `sonnet medium`。如果会话不接受该模型的该 effort,请让用户选择。

**(c) 展示角色并确认。** 展示每个角色及其值,并标出不在检测到的模型集合中的模型,标记为需要选择。同时列出第 2 步丢弃的每一行。询问是接受现状还是更改特定角色,提供检测到的模型,以及 `inherit-parent` 和 `auto`(二者的含义都是:该角色运行在父级聊天模型上)作为选项。对于 panel 角色(arena runners、architect runners、interrogate reviewers),其值是一个列表,每个条目运行一个 subagent,包括别名条目,因此列表长度决定了扇出数量。`arena cross-judge pool` 同样是列表,但 Arena 会从中选择一个值,其模型尽可能与父级模型不同。`swarm workers` 是每个 worker 的默认模型,除非 race 或 comparison 为每个分支指定了其他模型。

### 4. 校验

写入的每个模型都必须在检测到的集合中。`inherit-parent` 和 `auto` 始终通过。如果所选模型不可用,请停止并重新询问。

### 5. 写入规则

编写 `~/.claude/rules/pstack-models.md`,不加 frontmatter,然后是注释行、一行 `# budget`(包含所选标签及其目标 effort),以及每个角色一行,使用 poteto-mode 所用的相同标签。完整覆盖整个文件,使重新运行保持幂等。绝不要把角色行写入 `CLAUDE.md` 或 `AGENTS.md`。格式如下:

```
# pstack model configuration. One line per role. Delete a line to fall back to the skill default.
# `inherit-parent` or `auto` as a value: the role runs on the parent chat model (omit the Agent `model`). Alias entries in a panel list still count toward its fan-out.
# budget: high
feature, refactoring: sonnet high
bug-fix: sonnet high
perf-issue: sonnet high
hillclimb: sonnet high
judgment and prose: opus high
hardest tasks: opus high
how explorer: sonnet high
how explainer: opus high
why investigators: sonnet high
why synthesizer: opus high
reflect tooling: sonnet high
reflect judgment, divergent, synthesizer: opus high
arena runners: opus high, sonnet high
arena cross-judge pool: opus high, sonnet high
swarm workers: sonnet high
architect runners: opus high, sonnet high
interrogate reviewers: opus high, sonnet high
```

### 6. 确认

告诉用户规则已写入,并且它适用于新会话。重新运行此 skill 会更新它。

### 7. 提供验证 skill(可选)

检查项目是否已有驱动真实应用以提供证明的方式(一个 `verify-*` skill,或已有的 harness)。如果没有,提供一次:"want a project-local verification skill, so agents can drive the app the way a user does and prove changes work? I can generate one with /create-verification-skill." 如果用户同意,调用 `/create-verification-skill`(无论 pstack 安装在 workspace、用户目录还是插件中,都能解析到)。如果用户拒绝,则继续,不再推动。

### 8. 提供形式化验证 skill(可选)

检查项目是否已有形式化验证 skill(一个对代码运行模型检查器、证明或其他验证器的项目 skill)。如果没有,提供一次:"want a formal verification skill, so agents can prove a key property of your code holds in every case in scope, not only the cases tests reach? I can generate one with /create-formal-verification-skill." 如果用户同意,调用 `/create-formal-verification-skill`(无论 pstack 安装在 workspace、用户目录还是插件中,都能解析到)。如果用户拒绝,则继续,不再推动。
