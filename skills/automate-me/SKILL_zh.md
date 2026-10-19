---
name: automate-me
description: "当用户说 \"automate me\"、\"create/update/refresh my -mode skill\"、\"turn/capture my preferences or working style into a skill\"，或希望 agent 遵循用户的工作方式时使用。基于 create-skill 加 unslop 起草或修订个人 -mode skill，可选择从近期 transcript 中提取新证据。"
disable-model-invocation: true
---

# Automate me

在 Claude Code 中，遵循本 skill 之前，每个会话读取一次 [the Claude Code adapter](../../docs/harness/claude-code.md)。它把 pstack skills 中的 Cursor 名称映射到 Claude Code。

这是一个引导式流程，用于把用户的工作约定转化为 agent 会遵循的 skill。输出是一个为用户量身定制的 `-mode` skill（例如 `jay-mode`、`priya-mode`）。

本 skill 编排另外三个 skill：一个内联的挖掘步骤（见第 1 步）、Cursor 内置的 `create-skill`（编写），以及 **unslop** skill（文字规范）。它负责排序，不负责替代它们。

## 流程

### 0. 检查现有 skill

递归查找 `.cursor/skills/**/*-mode/SKILL.md` 和 `~/.cursor/skills/*-mode/SKILL.md`，并匹配用户的 handle。Mode skill 可以位于个人分类目录（`.cursor/skills/<handle>/`）中，而不只是顶层。如果存在，使用 `AskQuestion` 确认意图（除非用户已经说过"update my skill"或类似内容）：

- 更新现有 skill（重复运行时的默认选项）
- 从头开始（少见，执行前先询问原因）

更新模式会改变后续流程：
- 第 1 步只挖掘自 skill 上次编辑以来的历史（`git log -1 --format=%cI <path>`）。
- 第 2 步询问哪些内容发生了变化或缺失，而不是从零开始询问要捕获什么。
- 第 4 步原地编辑现有文件。保留用户未曾反驳的部分。修订有新证据支持的部分。仅为真正新增的规则添加新章节。

### 1. 挖掘用户历史

在并行扇出之前，先定位当前工作区的 transcript。系统提示会给出工作区的 `agent-transcripts/` 目录。只使用该路径。不要跨 `~/.cursor/projects/*/` 进行 glob。那样会跨越工作区边界，读取无关项目的私人聊天记录。

在该范围内调查近期的 agent 对话，寻找反复出现的模式。对历史的不同切片运行多个并行子 agent（例如最近 2-4 周，切分为 3 个切片，以保证每个切片有足够的素材）。根据 **poteto-mode** skill 的 Subagents 部分，为该任务选择每个子 agent 的推理等级以及快速或标准速度。每个切片的挖掘子 agent 从父级提供的工作区范围路径读取 transcript，查找下列信号，并返回一份简短的结构化模式列表，附带证据指针。默认值得查找的信号：

- 响应偏好（长度、语气、格式，"dumb it down"之类的纠正）
- 委派习惯（子 agent、模型、专门的工作流、并行度）
- 验证姿态（"完成"意味着什么，单元测试与实时复现，审阅者）
- 代码与文字规范（风格、引用的原则、lint/format 工具）
- 流程约定（worktree、commit、PR、审阅/合并工具）
- 元偏好（任务中途修复 skill、提议新 skill）

在提升某个信号之前先进行跨切片核对。在 2 个或以上切片中出现的模式是高置信度的。单独出现的信号较弱，通常会被舍弃。

### 2. 直接询问用户

挖掘会漏掉尚未出现的意图。使用 `AskQuestion` 工具（结构化多选），而不是让用户从零开始输入。

形式：一到两个问题，每个问题 4-6 个选项，类别问题使用 `allow_multiple: true`。先宽泛开始（"Which areas matter most?"），然后针对选中的领域用具体选项追问。结构化轮次之后，用一个自由格式的聊天问题捕获选项未覆盖的内容。

不要一次抛出 20 个问题。

### 3. 归类发现

将合并后的信号归入各个章节。常见章节（只使用适用的部分）：

- **Response style**：长度、语气、格式。
- **Autonomy**：无需询问时可以做多少事，MCP 工具的使用。
- **Understand first**：在界定或调查变更范围时应使用哪些 skill。
- **Subagents**：默认设置、并行度、模型与任务的匹配、专门的工作流。
- **Prose / code discipline**：原则、lint 工具、风格指南。
- **Review and verify**：复现姿态、验证 skill、实时测试工具。
- **Process**：git worktree、commit、PR、审阅/合并工具。
- **Skills**：skill 编写习惯、先修复 skill、提议新 skill。

**poteto-mode** skill 展示了这种结构。阅读它以掌握粒度。不要复制其内容。用户的规则与 poteto-mode 的规则并不相同。

### 4. 起草 skill

使用 Cursor 内置的 `create-skill` skill 编写本 skill。放置位置：

- 路径：保留现有 mode skill 的分类。对于新 mode，当仓库中已为该 handle 建立个人分类时，使用 `.cursor/skills/<handle>/<handle>-mode/SKILL.md`。否则，默认使用项目中的 `.cursor/skills/<handle>-mode/SKILL.md`（如果用户更倾向于个人 skill，则使用 `~/.cursor/skills/<handle>-mode/`）。
- Handle：用户的名字或其选定的标识符。
- Frontmatter 的 `description`：基于用户的名字 + `/<handle>-mode` + "work in their style" 触发，而不是基于 "write code" 或 "review PR" 之类的通用关键词。
- Frontmatter 格式：遵循 `create-skill` 的 YAML 规则。将 `description` 保持为单个 YAML 标量。当标点或换行需要时，为其加引号，或使用带缩进续行的 `description: >-`。
- Frontmatter 中默认设置 `disable-model-invocation: true`。只有当用户明确希望其 mode 在每一轮都生效时，才选择不设置。

### 5. 迭代文字

对每一行应用 **unslop** skill 和 `create-skill` 的写作指南。

将草稿展示给用户并收集反馈。预计会有多轮迭代。大胆删减。mode skill 不是操作手册。

### 6. 提交落地

在基于 main 的 worktree 中工作。提交 commit 并开启 PR。不要直接推送到 main。

## 护栏

- **不要对单次对话过度拟合。** 用户说过一次、又在另一次被推翻的偏好就是噪音。在将其固化之前，需要多次出现。
- **不要耍聪明。** 复述其他 skill 的内容、发明隐喻，或为 agent 读者撰写"诗意"的文字，都是有成本而无收益的。保持可操作性。
- **引用，而不是内联。** 用户依赖的其他 skill 应以路径引用的形式出现，而不是粘贴摘录。用户在别处维护的原则文档同理。
- **保持章节精简。** 只有当用户在该处有具体的、非默认的规则时，才添加章节。"Communicate clearly" 不是一个章节。"Short paragraphs. Tables when comparing options. Bullets only when items are genuinely parallel." 才是。
- **命名使用通用称谓。** 在祈使句中使用 "the user" 或 "the human"，不要使用作者的名字。
- **不要强行对称。** 如果用户没有值得写下的流程规则，就完全跳过 Process 章节。

## 评估

`-mode` skill 是主观输出。`create-skill` 风格的测试/迭代基准循环在这里没有用。与用户做一次直观检查：读起来像他本人吗？是否遗漏了什么？然后发布。

只有当 skill 的触发准确率在实践中成为问题时，才运行描述优化循环。

## 何时不使用

- 用户需要的是特定任务的 skill（而非工作约定）：只使用 `create-skill`，无需挖掘。
- 用户想捕获一个狭窄的工作流（例如 "how I write commit messages"）。那是一个普通 skill，而不是 mode skill。
