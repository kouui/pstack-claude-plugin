---
name: poteto-help
description: 引导用户完成 pstack 设置、/poteto-mode，并为任务选择合适的 skill、playbook 或 principle。输入 /poteto-help 加上问题即可使用。
disable-model-invocation: true
---

# Poteto 帮助

在 Claude Code 中，遵循本 skill 之前，每个会话先读一次 [Claude Code 适配器](../../docs/harness/claude-code.md)。它把 pstack skill 中的 Cursor 名称映射到 Claude Code。

回答用户关于 pstack 的问题，给出一个用户可以发送的 prompt，并链接答案所出自的文件。对于求助类问题，不要开始工作。用户问的是怎么做，而 pstack 运行会消耗真实的 token，所以应该让用户自己发送 prompt。

要求做事的消息，例如"用 pstack 修复这个 bug"，不是求助问题。阅读 [`poteto-mode`](../poteto-mode/SKILL.md)，按其流程完成工作，并提醒一次：Custom Mode 可以让它保持开启。

本文件把问题映射到存有答案的 skill 和指南页面。那些文件负责具体细节。引用之前先阅读你要链接的文件，并且当它与本映射不一致时，以它为准。这里的链接指向已安装的插件，用户可能无法打开，所以要给用户该文件的公开副本：`https://github.com/cursor/plugins/blob/main/pstack/` 加上它的路径。

## 弄清用户需要什么

根据消息和对话推断需求。已命名的情境，例如"哪个 skill 审查 PR？"，直接进入对应小节。如果需求仍不清楚，就问一个选择题，选项如下，然后只回答用户选中的那一节：

- 完成设置
- 用 `/poteto-mode` 开始一个任务
- 为某个情境选择 skill
- 修复出错的运行
- 把 pstack 变成自己的

检查那些会改变答案的状态，只在它确实有影响时提及：

- 如果没有 `~/.claude/rules/pstack-models.md`，说明该用户还没有运行 `/setup-pstack`，因此每个角色都使用默认模型。
- 如果项目中没有 `verify-*` skill 或其他应用 harness，说明 agent 没有脚本化的方式驱动应用。当问题与证明某项修改有效有关时，提及 `/create-verification-skill`。
- 如果项目中没有正式验证 skill，说明任何属性都无法超出测试覆盖的范围被检查。当问题与竞争条件、不变量或其他测试很少触及的情况有关时，提及 `/create-formal-verification-skill`。当它已经存在，并且它覆盖的代码在它记录的最后同步提交之后发生了变化，如果问题涉及这些代码，就提及 `/maintain-formal-verification-skill`。

当缺少模型规则且这很重要时，询问用户是否想现在为每个角色选择模型以及推理预算。以下情况下这很重要：用户是新手、问题与设置或成本有关，或答案取决于运行的是哪些模型。每个聊天最多询问一次。如果需求也不清楚，就把两个问题一起问。提供两个选择：

- 现在：给用户一条要输入的 `/setup-pstack`，同时也回答他们的问题。
- 稍后：回答他们的问题，并加一行说明，在他们运行 `/setup-pstack` 之前，每个角色都保持默认模型。

## 完成设置

1. 在聊天中用 `/add-plugin pstack` 安装，或从侧边栏的 Customize 安装。
2. 运行 [`/setup-pstack`](../setup-pstack/SKILL.md)。它会询问推理预算，为每个角色映射一个模型，并写入一条规则。该规则适用于新聊天。然后，除非项目已经有对应 skill，它会各提供一次为项目生成验证 skill 和正式验证 skill 的选项。
3. 用 `/poteto-mode`、一个目标，以及一个可能通过也可能失败的检查项，开始一个真实任务。

在用户调用某个 skill 之前，安装不会改变任何东西。只有 `/setup-pstack` 会根据用户的话语加载。[README](../../README.md) 和[指南第 1 页](../../docs/guide/01-setup.md)有详细说明。根据 [`references/prompting.md`](references/prompting.md)，主动提出帮用户措辞他们的第一个 prompt。

如果用户担心成本，说明 token 花在哪里，以及如何少花。pstack 会在 subagent 和审查小组上额外消耗 token。重新运行 `/setup-pstack`，选择更小的预算或更便宜的模型。被设置为 `auto` 或 `inherit-parent` 的角色会使用聊天所用的模型运行，当聊天使用 Auto 或更便宜的模型时，这样能节省 token。更短的面板列表会运行更少的 subagent，每个条目对应一个。只在需要严谨性的工作中使用 `/poteto-mode`。

pstack 是为 Cursor 构建的，它也可以通过 `/plugin marketplace add cursor/plugins`，然后执行 `/plugin install pstack@cursor-plugins` 安装到 Claude Code 中。在那里，每个命令都要加上 `pstack:` 前缀，例如 `/pstack:poteto-mode`。[Claude Code 适配器](../../docs/harness/claude-code.md)负责映射 Cursor 名称，而需要仅限 Cursor 功能的步骤，例如 Bugbot 或 Automations，会被跳过，并附一行说明。

其他工具可以读取这些 skill，因为它们使用 Agent Skills 格式。但大多数工作流 skill，包括 `/poteto-mode`、`/how`、`/why` 和 `/teach`，会启动带有按角色模型的 Cursor subagent，而 Custom Mode 和 `/loop` 都是 Cursor 功能，因此这些部分在其他工具中可能无法工作。

## 用 `/poteto-mode` 开始任务

`/poteto-mode` 会把任务匹配到一个 playbook，把 playbook 的步骤复制到待办列表中，并在步骤需要时运行其他 skill。被跳过的步骤会以 `skip: <reason>` 的形式留在列表中。好的 prompt 会说明目标以及如何判断任务已完成。它不列出 skill，因为手写的顺序往往会丢掉或打乱 playbook 原本会保留的步骤。在帮用户措辞 prompt 之前，阅读 [`references/prompting.md`](references/prompting.md)。[指南第 2 页](../../docs/guide/02-poteto-mode.md)有示例。

`/poteto-mode` 是否保持开启，取决于用户如何启动它：

- 在 `/poteto-mode` 上按 Enter，会把该 skill 附加到单条消息上。随着对话推进，它会逐渐失效。
- Mac 上按 Option+Enter，或 Windows 上按 Alt+Enter，或从 skill 条目中选择 Use as Mode，会把它变成 Custom Mode。它会在每一轮都保留在上下文中，直到用户退出该模式，并且不会出现在闲聊的轮次中。
- Cursor 的文档列出了 Agents Window 和 CLI 中的 Custom Mode。在其他地方，每个新任务都要以 `/poteto-mode` 开始。

提到这一点时，链接 [Cursor 的 skills 文档](https://cursor.com/docs/skills)。在聊天中途，说"新任务"会让该模式匹配一个新的 playbook。`/poteto-mode` 已经为其 playbook 步骤所启动的 subagent 使用 `poteto-agent`。要让你自己的 subagent 具有相同风格，请用 `subagent_type: "poteto-agent"` 启动它。

## 选择 skill

默认答案是 `/poteto-mode`，当其步骤需要时，它会运行大多数其他 skill。当用户想要的内容比 playbook 提供的更多或更少时，直接点名某个 skill。在推荐之前先阅读该 skill，并给出一个示例 prompt。

| 用户想要 | Skill |
|---|---|
| 以严谨的方式完成任何非平凡任务 | [`/poteto-mode`](../poteto-mode/SKILL.md) |
| 在开始前确认 agent 理解了他们的目标和问题 | [`/in-your-own-words`](../in-your-own-words/SKILL.md) |
| 在开始前把任务提得更高：更专业、更关注结果、不过度规定 | [`/sharpen`](../sharpen/SKILL.md) |
| 了解代码现在如何工作，或新代码应该放在哪里 | [`/how`](../how/SKILL.md) |
| 了解代码为什么是这样的形态，或某个数字从哪里来 | [`/why`](../why/SKILL.md) |
| 以平实的方式理解某次修改或某个子系统 | [`/teach`](../teach/SKILL.md) |
| 回顾自己近期在某个主题上的工作 | [`/recall`](../recall/SKILL.md) |
| 用平实的文字和小型可视化，了解自上次消息以来本聊天中发生了什么变化，并且每个等待用户处理的事项都会列出，仅凭回复即可决定 | [`/ketchup`](../ketchup/SKILL.md) |
| 了解一个小 diff 可能在其自身之外破坏什么 | [`/blast-radius`](../blast-radius/SKILL.md) |
| 在跨越函数边界的代码之前，确定类型和模块形态 | [`/architect`](../architect/SKILL.md) |
| 对同一个简报获得多次尝试，并合并为最佳版本 | [`/arena`](../arena/SKILL.md) |
| 将并行检查分片运行，或以云端 agent 的方式竞相执行 worker | [`/swarm`](../swarm/SKILL.md) |
| 让不同模型审查 diff，并尝试将其破坏 | [`/interrogate`](../interrogate/SKILL.md) |
| 当存在廉价的本地测试时，以测试先行的方式修复 bug | [`/tdd`](../tdd/SKILL.md) |
| 将 TypeScript 规则应用于 `.ts` 或 `.tsx` 工作 | [`/typescript-best-practices`](../typescript-best-practices/SKILL.md) |
| 在审查前去除注释，使用未编写这些注释的审查者 | [`/no-comments`](../no-comments/SKILL.md) |
| 清除散文中的 AI 痕迹 | [`/unslop`](../unslop/SKILL.md) |
| 按标准撰写文档、RFC、README、PR 描述或 commit 信息 | [`/technical-writing`](../technical-writing/SKILL.md) |
| 用平实的文字再听一遍上一条回复 | [`/bro`](../bro/SKILL.md) |
| 为 agent 提供脚本化的方式来驱动应用并证明行为 | [`/create-verification-skill`](../create-verification-skill/SKILL.md) |
| 让验证 skill 及其功能地图与应用重新保持一致 | [`/maintain-verification-skill`](../maintain-verification-skill/SKILL.md) |
| 正式验证测试很少触及的竞争条件、不变量或其他属性 | [`/create-formal-verification-skill`](../create-formal-verification-skill/SKILL.md) |
| 在其覆盖的代码发生变化后，让正式验证重新与之保持一致，并判断失败是漂移还是 bug | [`/maintain-formal-verification-skill`](../maintain-formal-verification-skill/SKILL.md) |
| 在报告或据此行动之前，核实某个性能数字 | [`/benchmark-checklist`](../benchmark-checklist/SKILL.md) |
| 运行大型或跨领域的修改，或离开后要审查的修改 | [`/figure-it-out`](../figure-it-out/SKILL.md) |
| 在运行过程中保存决策日志，并在事后回顾 | [`/show-me-your-work`](../show-me-your-work/SKILL.md) |
| 为每个角色选择模型和推理预算 | [`/setup-pstack`](../setup-pstack/SKILL.md) |
| 把自己的工作习惯转化为个人 mode skill | [`/automate-me`](../automate-me/SKILL.md) |
| 把已完成任务的经验转化为 skill 编辑 | [`/reflect`](../reflect/SKILL.md) |
| 阻止 agent 在此仓库中重复同样的错误 | [`/correct`](../correct/SKILL.md) |
| 构建一个按钮通过 webhook 唤醒 Grok Bot 的页面 | [`/make-bot-ui`](../make-bot-ui/SKILL.md) |
| 了解如何在 pstack 中找到方向 | `/poteto-help` |

如果此目录旁边的某个 skill 目录未出现在表格中，读取它的 frontmatter，并根据其 description 进行路由。`principle-*` 目录在下面的 principles 中说明。

难以取舍的情况：

- `/how` 解释代码做了什么。`/why` 解释原因。`/teach` 会运行其中一个或两个，并以平实的方式解释结果。
- `/arena` 会给每个 worker 相同的简报，并合并最佳部分。`/swarm` 将工作拆分为分片或竞赛，并返回一份报告。
- `/architect` 在确定设计后紧接着实现。添加"with checkpoint"可以在编写代码前审查设计。
- `/interrogate` 审查 diff。`/blast-radius` 查找 diff 之外的破坏，并证明使该修改安全的那一个事实。
- `/recall` 在近期聊天之间重建上下文。`/ketchup` 只涵盖用户上一条消息之后的本聊天。恢复某个特定聊天或分支属于 Session pickup playbook。
- `/figure-it-out` 设计一次严谨的运行。Orchestrate playbook 运行跨越数天和多个 PR 的程序。Autonomous run playbook 把一个任务推进到完成条件。

不在 pstack 中的内容：

- `/deslop`、`control-cli` 和 `control-ui` 随 `cursor-team-kit` 插件提供。
- `/loop` 和 `/create-skill` 是 Cursor 内置功能。
- pstack 没有 `/orchestrate` skill。Orchestrate 是 `/poteto-mode` 的一个 playbook。如果斜杠菜单显示 `/orchestrate`，说明是另一个插件提供的。

## Playbook 与 principle

Playbook 是 `/poteto-mode` 内部的步骤列表，而不是 skill，因此它们没有斜杠命令。在 `/poteto-mode` 中，描述任务会选中某个 playbook，而以下短语会直接点名某个 playbook：

- "babysit this pr" 或 "check on pr 123" 会运行 Babysit。它会把 PR 推进到可合并状态，然后停在那里。除非用户要求 merge、land 或 ship，否则它不会合并。
- "land the stack" 会运行 Shipping。
- "take over this branch" 会运行 Session pickup。
- "pause safely" 会运行 Pause safely。
- "full autopilot on this queue" 会运行 Autopilot-full。"stack them, don't ship" 会运行 Autopilot-stack。
- "run the eval playbook" 会运行 Eval。

没有 `/poteto-mode` 时，"babysit this pr" 之类的短语可能会改为启动 Cursor 自带的同类任务 skill。[`poteto-mode`](../poteto-mode/SKILL.md) 的 Playbooks 小节列出了每个 playbook 以及它适用的场景。[指南第 6 页](../../docs/guide/06-verify-and-ship.md)涵盖打开、照看和合并 PR。

pstack 没有规划 skill。Cursor 的 Plan Mode 可以与之配合使用。对于跨越多个阶段或堆叠 PR 的工作，要求 `/poteto-mode` 制定计划，会运行[多阶段计划 playbook](../poteto-mode/playbooks/multi-phase-plan.md)，它会写出计划但不实现。对于设计问题，Prototype playbook 或 `/architect` 会先用代码把问题定下来。

Principle 是 `/poteto-mode` 读取并在回复中引用的单条规则 skill。用户很少直接调用它们。他们改为用名称来引导，例如"apply prove it works. show me the real output."。输入 `/principle-<name>` 仍可按需加载某一条。[指南第 8 页](../../docs/guide/08-principles.md)列出了它们。

## 修复出错的运行

| 症状 | 修复方法 |
|---|---|
| 该模式在几轮之后停止生效 | 它是用 Enter 启动的。以 Custom Mode 的方式启动它，或者每个任务都以 `/poteto-mode` 开始。 |
| 一个问题被当作上一个任务的下一步来处理 | 说"new task"，或说明这一轮不需要该模式。 |
| 新的模型选择没有效果 | `/setup-pstack` 的规则适用于新聊天。开始一个新聊天。 |
| 运行成本高于预期 | 参见"完成设置"下的成本段落。 |
| 某个 skill 没有自动加载 | 只有 `/setup-pstack` 会根据用户的话语加载。其他 skill 在用户输入它们时加载，或在 `/poteto-mode` 运行它们时加载，而它不会运行每一个 skill。 |
| 并行 agent 互相覆盖了内容 | 为每个 agent 提供自己的 worktree，或将它们作为云端 agent 运行，每个云端 agent 都有自己的机器。 |
| 一次夜间运行有进展，但没有完成任何事 | `/loop` 需要一个可能通过也可能失败的检查，而不是一个时长。参见[指南第 7 页](../../docs/guide/07-overnight.md)。 |
| 回复因构建变绿而声称成功 | 要求提供真实的命令、流程、存储值或性能剖析结果。这就是 prove-it-works principle。 |

对于偏离方向的运行，[`references/prompting.md`](references/prompting.md) 提供了一行式的纠偏提示。[指南第 10 页](../../docs/guide/10-recipes-and-pitfalls.md)有更多陷阱以及值得借鉴的做法。

## 把 pstack 变成自己的

- [`/automate-me`](../automate-me/SKILL.md) 根据用户自己的历史，起草一个个人 mode skill，供与 `/poteto-mode` 一起使用。
- 一次会话之后，[`/reflect`](../reflect/SKILL.md) 会把其经验转化为用户批准的 skill 编辑。
- `/poteto-mode write a skill for <workflow>` 运行编写 playbook。Eval playbook 会以盲测方式测试 skill 修改。
- 在其自身的 PR 中修复行为异常的 skill，而不是在它出错的功能工作中修复。

[指南第 9 页](../../docs/guide/09-make-it-yours.md)涵盖了以上每一项。

## 回复

先给出答案。最多给出一个代码块中的示例 prompt，如果合适，可根据 [`references/recipes.md`](references/recipes.md) 改编，然后附上该文件的链接。除非用户要求完整地图，否则保持简短。
