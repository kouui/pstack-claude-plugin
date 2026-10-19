---
name: why
description: "用于 'why does X work this way'、'why we picked Y'、设计依据、回归问题、事后复盘或有数据支撑的阈值。会发现可用的 MCP，并在单条消息中并行查询每个证据类别（源码控制、问题追踪器、长文档、实时团队聊天、基础设施可观测性、错误追踪、产品分析数据仓库），然后返回一份带引用的关于决策和权衡的解读。how 用于运行时行为。"
disable-model-invocation: true
---

# Why

在 Claude Code 中，遵循本技能前每个会话读取一次 [Claude Code 适配器](../../docs/harness/claude-code.md)。它把 pstack 技能中的 Cursor 名称映射到 Claude Code。

调查代码背后的动机和意图。配套的 `how` 技能回答代码做什么以及如何工作。`why` 回答哪些力量促成了它的当前形态。

下面每个 spawn 都命名了 `pstack-models.md` 规则中的一个角色行及其默认值。将 `model` 设为该行的值；如果规则或该行缺失，则使用默认值。当值为 `auto` 或 `inherit-parent` 时，不要设置 `model`。如果 Task 工具拒绝某个 slug，则使用默认值并说明这一点。如果它连默认值也拒绝，则使用其错误信息中同一系列最接近的有效 slug。按照 **poteto-mode** 技能的 Subagents 部分，为每个子代理选择合适的推理级别以及快速或标准速度。

每次 spawn 都是 `generalPurpose`，且 `readonly: false`（代理模式）。**不要使用 readonly/Ask 模式。** 它会剥离 MCP 访问权限，从而禁用基于 MCP 的调查者以及综合者的引用抽查。调查者仍然不应写入任何内容。

## 操作姿态

以**谨慎、小心且精确的调查者**身份工作。诚实说明你所知与你所推断的内容。完整的置信度框架和措辞指南请阅读 `references/epistemics.md`。综合者必须遵循它。

## 第 1 步. 理解目标和问题

**目标**通常是一段代码、一种模式、一个功能或一个被命名的设计决策。**问题**通常是设计依据、权衡、促成某个边界情况的动机、外部约束、死代码，或广泛的历史梳理。

如果目标含糊（"why do we do it this way?" 没有明确指向），请根据对话上下文（打开的文件、最近的编辑、光标位置、刚讨论过的内容）做出最佳猜测。简要说明你的理解，以便用户在你理解有误时予以纠正，然后继续。

## 第 2 步. 建立代码锚点

在派出调查者之前，在当前上下文中构建种子上下文：文件路径和行范围、关键符号（函数、类、常量）、最近触及目标的几次提交、来自合并提交的 PR 编号（主题行中的模式 `(#1234)`），以及任何关联的工单 ID。运行 `git blame -L <start>,<end> <file>` 获取最后修改提交，运行 `git log --follow -p -- <file>` 获取跨重命名的完整历史及补丁，运行 `git log --oneline -20 -- <file>` 获取可见 PR 编号的近期提交，运行 `git log -1 --format=%B <commit>` 提取 PR 编号。然后拉取任何实质性提交的 PR 正文和讨论：`gh pr view <number> --json title,body,author,createdAt,mergedAt,labels,closingIssuesReferences,comments,reviews`。此锚点会发送给每个调查者和综合者。

## 第 3 步. 派出并行调查者

**默认进行完整的并行调查。** 目标是完整的覆盖地图，而不是最小化的地图。记录空结果，不要跳过搜索。

### 发现

在派出调查者之前，从 Cursor 环境中列出可用的 MCP。如果存在可用工具映射则使用它。否则检查 Cursor 为已启用 MCP 服务器暴露的 `mcps/` 目录。

将每个可用的 MCP 映射到一个证据类别：

1. 源码控制历史
2. 问题 / 工单追踪器
3. 长文档
4. 实时团队聊天
5. 基础设施可观测性
6. 错误 / 异常追踪
7. 产品分析数据仓库

源码控制始终通过 git 和 `gh` 可用。对于其余六类，使用 MCP 名称、服务器说明、工具名称和资源描述符进行分类。如果某个 MCP 可能属于多个类别，则选择与其主要证据相匹配的类别。将有歧义的情况记录在覆盖地图中。

对于每个有匹配 MCP 的类别派出一个调查者，全部在单条消息中派出以便并发运行，每个调查者只负责恰好一个工具或 MCP。不要让一个代理负责多个 MCP。`model`：`why investigators` 行，默认 `sonnet high`。

每个调查者的提示词是填写好的 `references/investigator-prompt.md`，外加：
- 所选 MCP 的类别操作手册 `references/sources/<source>.md`，改编自 `references/source-playbook.md` 中的示例
- 如果目标代码看起来具有防御性（空值检查、重试逻辑、超时处理、速率限制、功能开关、出口守卫、OOM 处理程序），则加入 `references/sources/incident-postmortem.md`
- 代码锚点和用户的原始问题

### 调查者名单，每个可用证据类别一个

下面每个类别都能独特地揭示一种形式的 "why"，列在 "Best at" 之后。用它来了解预期会返回什么，在某个类别返回空结果时指出其缺口，并且仅在极少数可证明无关的情况下，用它来证明跳过是合理的。

1. **源码控制**（git 历史、用于 PR 的 `gh`、代码注释、测试）。始终派出，是唯一有保证的来源。Best at *在评审过程中捕获的实现时理由*。
2. **问题 / 工单追踪器**（例如 Linear、Jira、GitHub Issues、Plane、Shortcut MCP）。Best at *产品或业务的驱动因素*。当 why 来自工程之外时最有力。
3. **长文档**（例如 Notion、Confluence、Google Docs、Coda MCP）。Best at *在变成代码之前写成的长篇设计依据*。
4. **实时团队聊天**（例如 Slack、Discord、Microsoft Teams、Mattermost MCP）。Best at *从未进入文档的实时讨论*。当源码控制、工单和文档的纸面记录很薄弱时尤为重要。
5. **基础设施可观测性**（例如 Datadog、New Relic、Honeycomb、Grafana、Splunk MCP）。Best at *促成该代码的基础设施和运行时现实*。当目标对基础设施信号（超时、重试、速率限制、断路器）作出反应时最有力。
6. **错误 / 异常追踪**（例如 Sentry、Rollbar、Bugsnag、Airbrake MCP）。Best at *促成防御性或纠正性代码的具体异常和错误轨迹*。对 catch 块、空值守卫、类型检查、重试及其他防御手段最有力。
7. **产品分析数据仓库**（例如 Databricks、Snowflake、BigQuery、ClickHouse、dbt、Redshift MCP）。Best at *塑造该代码的产品和数据现实*。对于功能开关控制的代码、实验驱动的上线、数据迁移以及 "this number came from where" 之类的问题最有力。

### 何时跳过调查者

只能附带**明确且书面的理由**，并写入最终的 Sources Consulted 中。有两种有效理由：

- **该环境中没有该类别可用的 MCP**。将其标记为缺口而非选择："Real-time team chat skipped. No matching MCP available, so the conversational record was not searchable."
- **来源明确无关**，而不仅仅是"可能无关"。门槛很高："Error / exception tracking skipped. Target is a build-time script with no runtime code path."

如果你的范围评估表明这是一个单次提交的简单目标，且 PR 描述已经包含完整答案，只有在确认所有七个可用类别的搜索都是多余的之后，才可以直接内联回答。需要明确说明这一点。这种情况应当很少见。

## 第 4 步. 综合

派出一个综合者。`model`：`why synthesizer` 行，默认 `opus high`。它获得：
1. 调查者的发现，包括任何空结果以及带理由跳过的类别
2. 代码锚点
3. 用户的原始问题
4. `references/epistemics.md` 中的认知框架
5. `references/synthesizer-prompt.md` 中的综合者提示词模板

## 第 5 步. 呈现

将综合者的输出呈现给用户。你可以为了清晰而轻度编辑，或补充对话中的上下文，但**不要改写置信度措辞**。

## 输出格式

结构采用 `references/synthesizer-prompt.md` 中的结构：The Question、The Code in Question、What We Found、What We Can Reasonably Infer、Competing Hypotheses、What We Don't Know、Sources Consulted、Confidence Summary。可根据需要调整，但要保持置信度的区分，并且 Sources Consulted 中每个调查者占一行，包括那些没有返回任何内容或被跳过的调查者，并注明原因。

在 Sources Consulted 之后，如果用户的 `why` 问题是修改此代码的前期工作，则将谱系发现转换为一组 Preserve / Change / Avoid / Risk 约束，用于规划该修改。

## 应避免的常见失败模式

- **近因偏差**。假定最近的提交是权威的。当前形态往往是许多早期决策累积的结果。要追溯。
