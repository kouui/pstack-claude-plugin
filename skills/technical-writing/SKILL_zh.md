---
name: technical-writing
description: "分层技术写作标准：Diátaxis 结构、Google 开发者风格句子、STE 指令规则、Global English 句法。用于 /technical-writing，或在编写或审阅文档、RFC、readme、PR 描述或 commit 消息时使用。"
disable-model-invocation: true
---

# Technical writing

在 Claude Code 中，遵循本 skill 之前，每个会话读取一次 [the Claude Code adapter](../../docs/harness/claude-code.md)。它把 pstack skills 中的 Cursor 名称映射到 Claude Code。

写作要让一个疲惫的工程师第一次读就能看懂。四层全部适用。

以下三条规则高于各层：

- **删掉所有不起作用的词。** 如果去掉某个词，句子意思不变，就删掉它。"In order to" 写成 "to"。"It is important to note that" 整句删掉。
- **用简短、日常的词。** 用 "Use"，不用 "utilize"。用 "Help"，不用 "facilitate"。用 "Do"，不用 "perform"。长词必须靠精确性换取它的长度。
- **当某条规则让句子变差时，换一种方式改写句子，或者不改。** 一个完全遵守每条规则、读起来像机器写的句子，同样是失败的。

代码库就是词表。写真实的符号名、文件名、flag 或命令名，不要用同义词或描述来代替。

不要发明行话。使用开发者会口头说出的词："move"、"delete"、"a budget that only decreases"，而不是 "evacuate"、"ratchet" 或 "endgame"。命名的模式可以用，但文档第一次出现时必须说明它的含义。如果发现新的问题用语及其替代写法，请在你的回复中把它作为对 `unslop` 的 abstract-metaphor 规则的补充提出，并附上 diff。不要编辑那个 skill。

## 变化节奏

一份文档即使遵守了所有层，读起来仍可能像机器写的：每个句子都被剪得很短，没有任何观点，也没有任何具体内容。

- 有意混合句子长度。短句用来突出一个要点。长句用来承载一个事实，以及它的条件或后果。
- 每句只表达一个想法，并不意味着每句长度都一样。拆开承载两个想法的句子。保留只承载一个想法的长句。
- 在文体允许的情况下，要有自己的观点。讲解类内容要权衡利弊，并说明你如何看待这些权衡，而不是只列出优缺点。参考类内容保持干巴。
- 要具体，不要空洞。不要写 "schema changes can cause issues"，而要写 "a column rename fails the build"。

## 先选定模式（Diátaxis）

一份文档只用一种模式。两个问题决定模式：内容是指导行动（做）还是指导理解（想），以及它服务于学习还是工作。

- 行动 + 学习：**tutorial**。
- 行动 + 工作：**how-to**。
- 理解 + 工作：**reference**。
- 理解 + 学习：**explanation**。

可以对整份文档或某一个句子使用这个罗盘。

**Tutorial：通过动手学习。** 让学习者成功是你的责任，而不是他们的。开头说明学习者将构建什么，而不是他们将 "learn" 什么。每一步都要尽早、频繁地产生可见的结果。告诉他们应该看到什么：预期输出、提示词的变化、日志行。把解释压缩为一个分句和一个链接。保持具体。以 "we" 的口吻写，在命令中使用："First, do x. Now, do y."

**How-to：达成目标的步骤。** 解决读者遇到的问题，而不是机器能执行的操作。假定读者已具备能力，省略教学内容。只写行动：不要离题，不要写背景，不要为了完整性而完整。需要时链接到相关内容。允许分支和判断："If you want x, do y."。以任务命名指南："How to calibrate the radar array"，而不是 "Radar array calibration"。

**Reference：供查阅的事实。** 只描述。不指示，不劝说，不发表意见。语气干脆、完整、确定。陈述事实、选项、限制和错误时不加任何模糊措辞。让文档结构与所描述事物的结构保持一致，使代码和文档可以对照导航。把内容放在读者期望找到的位置。能从代码生成的，尽量从代码生成。

**Explanation：理解与原因。** 围绕一个有边界的主题，脱离产品也能读懂。每个标题都应能容纳隐含的 "About..." 前缀。以一个真实的 why 问题为锚点。提供上下文：设计决策、历史、约束、替代方案。只有这里允许表达观点，其他模式都不允许。

不要混用模式：tutorial 中不放参考表格，reference 中不放 tutorial 式的手把手指导，how-to 中不展开争论。应拆分并互相链接。

## 写句子要面向读者（Google 开发者风格）

- 用 "you" 称呼读者，使用现在时。"Will" 只用于确实会在以后发生的事情。
- 说明谁做什么："the compiler checks"，而不是 "is checked"。只有在执行者未知或无关紧要时，才使用被动语态。
- 把指令写成命令："Click Submit."。平实地陈述事实。不要写 "should be done"。
- 先写条件，再写指令："To delete the document, click Delete."。
- 常见情况放在前面。例外情况放在后面。
- 说话像一位懂行的朋友。不用流行语，不用比喻，指令中不用 "please"，而且在操作步骤中绝不用 "simply"、"easy" 或 "quickly"。如果真的很简单，读者就不会来看这篇文档了。
- 不要预先宣布（"we will soon support..."），也不要让连续的句子以相同的短语开头。
- 用说明去向的文字做链接：页面标题或简短描述。绝不用 "click here"。与其放一个指向外部的链接，不如在本页写一句上下文。
- 标题要点明要点，而不仅是主题（用 "Pick the mode first"，而不是 "Modes"）。使用句子大小写。任务标题是不带修饰的动词短语（"Create an instance"）。概念标题是名词短语。每页只有一个 h1，不跳级。
- 顺序性内容用编号列表，其他内容用项目符号列表。用一个完整的句子引出列表。列表项保持结构平行。
- 代码放在代码字体中。UI 元素加粗。使用串列逗号。删除 "etc."，并在前面说明列表不完整。

## 让语句逐条可读（STE 规则）

- 每句只有一条指令。其他地方每句只有一个想法。
- 超过约 20 个词的指令要拆分，超过约 25 个词的其他句子也要拆分。
- 把警告或条件放在它所约束的步骤之前："If hot oil touches your skin, injuries can occur."。
- 保留 "the" 和 "a"："Remove backup file" 有两种读法。"Remove the backup file" 只有一种读法。
- 每个词只有一个含义和一个用途，并始终保持。如果 "check" 表示检查，就不要再用它表示抑制。
- 每个动作只用一个词，并坚持使用：统一用 "start"，不要这里用 "start"、那里用 "initiate"。
- 把操作步骤写成直接命令，绝不写成叙述，也不用被动语态："Install the component"，而不是 "the component must be installed"。
- 能避免的地方就避免 "-ing" 词。

## 不让句子有两种读法（Global English）

- 把 "only" 和 "not" 这类词放在它们所修饰的词旁边："only fails on growth" 和 "fails only on growth" 表达的意思不同。
- 拆开过长的名词串："the proto import budget check script" 改为 "the script that checks the proto-import budget"。
- 让每个 "it"、"they" 和 "this" 只指向一个明显的对象。有疑问时重复名词。绝不用 "this" 或 "which" 指代整个分句。
- 不要省略动词："Phase 1 moves the converters and Phase 2 the runtime" 使 Phase 2 缺少动词。补上动词。
- 保留表示结构的小词，比如 "Ensure that the switch is off" 中的 "that"。绝不为了减少词数而牺牲清晰度。
- 当两者是两个不同事物、重复冠词能防止误读时，在并列结构中重复冠词："the client and the host"，而不是 "the client and host"。
- 当句子可以有两种分组方式时，说明 "and" 或 "or" 连接的是哪些部分。"Both...and"、"either...or" 和 "if...then" 是免费的消歧工具。
- 用句号，不用分号。把破折号替换为新的句子。
- 括号中的文字必须是完整的语法单位，或者自成一句。绝不用 "(s)" 构成复数。
- 不用斜杠：写 "a, b, or both"，不写 "a/b" 或 "and/or"。
- 每个事物在各处都只用一个名称。如果文档对同一事物一会儿写 "the gate"、一会儿写 "the ratchet"、一会儿写 "the budget check"，就等于教了读者三种东西。没有变化的地方不要随意换词。
- 不用习语、口语、拉丁缩写和隐喻。

## 语气与仓库细节

- 对本 skill 涉及的每份文档都应用 **unslop** skill。该 skill 负责 slop 模式目录：AI 词汇、冗余填充、模糊措辞、格式痕迹。
- PR 描述和 commit 消息同样属于写作。除 Diátaxis 外，其他各层都适用于它们。PR 正文是一份审阅者在一分钟内能读完的简报。不要粘贴 swarm 日志、SHA 列表或指标表格，而是链接到它们。
- 产品界面文案不属于文档。这类文案请使用你的产品文案指南。
- 代码片段使用 tab 缩进。写真实路径。每个计数或目录树的说法，在落地该变更的 commit 中都必须为真，并附上重新生成它们的命令。

## 完整示例

修改前：

> Configuration of the proto import ratchet budget script parameters is performed via budget.json. Note that it's important to remember that running with --write, which updates the committed budget to reflect the current count, should only be done when lowering it. If exceeded, CI fails.

修改后：

> `budget.mjs` reads the committed budget from `budget.json` and counts the files that import protos. If the count exceeds the budget, CI fails. Run `budget.mjs --write` only to lower the budget.
