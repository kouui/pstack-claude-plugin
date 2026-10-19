---
name: show-me-your-work
description: "为长时间运行或无人值守的工作保留可供复核的决策轨迹：一个 TSV 日志，每个决策一行（做了什么、为什么、证据、结果）。默认保存在本地；当复核者需要轨迹来信任结果时再提交。适用于 /show-me-your-work、自主或多阶段运行，以及人离开后需要复核的工作。"
disable-model-invocation: true
---

# 展示你的工作

在 Claude Code 中，每个会话开始时先阅读一次 [Claude Code 适配器](../../docs/harness/claude-code.md)，然后再遵循本技能。它将 pstack 技能中的 Cursor 名称映射到 Claude Code。

只保留一份权威日志。

## 格式

一个 TSV 文件，每个决策一行。单元格保持单行。证据是指针，而不是散文。

复制 `references/decision-log-template.tsv`（表头行）以开始一份干净的日志。列如下：

- **ts.** ISO8601 时间戳。
- **phase.** 阶段或工作流。
- **decision.** 所选择或所完成的事项，一行。
- **why.** 用平实的语言说明原因。如果是某条原则促成了该决策，直接平实地说明，不要使用术语标签。
- **evidence.** 证明该决策的链接或路径：commit SHA、PR 编号、`file:line`，或产物、追踪记录、截图路径。绝不写成段落。
- **result.** 结果或断言状态：`tests green`、`reverted`、`pixel-diff 0`、`INCONCLUSIVE`、`open`。

下面是一个示例，用平实的语言写成，便于复核者一眼看懂。

```
ts	phase	decision	why	evidence	result
2026-05-24T09:02:00Z	frame	counted the work first, about 100 components and roughly 75 hours	wanted to know the size before starting a long run	commit 3a9f1c2	found 5 things to sort out before starting
2026-05-24T09:40:00Z	harness	took screenshots of the old version before changing anything	so we can compare old against new and catch any visual change	scripts/snapshot.sh, baseline/	saved 120 reference screenshots
2026-05-24T11:15:00Z	widget	moved the widget styles over without changing how it looks	keep the change small and the result identical	commit 7c21e0a, pixel-diff 0	looks identical, tests pass
2026-05-24T12:30:00Z	widget	threw out a helper's work because its screenshots were blank	checked the real files instead of trusting its summary	worktree reset	reverted, tightened the instructions for next time
```

## 记录一行

按照你向同事解释自己做了什么的方式来写每条记录。使用平实的词语、具体的动作，不要用 AI 腔或抽象术语（**unslop** 技能同样适用于日志文本）。

使用辅助脚本 `scripts/log.sh <logfile> <phase> <decision> <why> <evidence> <result>`。它会为 `ts` 盖上时间戳，首次使用时写入表头，去除多余的制表符/换行符，并在以 `=`、`+`、`-` 或 `@` 开头的单元格前加上单引号前缀。直接用 `printf` 追加一行也可以，但如果单元格内容来自生成文本或用户提供的文本，要注意同样的字符问题。

只记录决策点和检查点，而非每一个动作：选定的分支、附带验证结果的完成单元、附带触发原因的转向或回滚、出现的阻塞项、修复的门禁。对于循环运行，每次迭代记一行。跳过琐碎和不言自明的内容。

一次运行就是一次智能体对话，包括其后续轮次以及对其的任何总结。接手、替换智能体或新开聊天都开始一次新的运行。当某次运行向已有行的日志追加内容时，其第一行的阶段为 `start`；在另一次运行的 `start` 行之后的第一行，阶段也为 `start`。因此，在后续轮次中回到日志的运行，会先阅读日志的最后几行，看看其间是否有其他运行写入过。`start` 行会注明它之前、非本次运行所写的行的 `ts` 范围，其证据会注明本次运行，例如其智能体 ID。`start` 阶段只能用于此目的。

## 存放位置

默认情况下，日志是工作产物，不提交。将其保存在工作目录中的 `decisions.tsv`，或当多项工作同时进行时保存为 `.audit/<task-slug>.tsv`，并将其排除在 git 之外。

只有当工作足够有野心，以至于复核者需要轨迹才能信任结果时，才提交该日志。

## 规则

- 只追加。错误的决定通过新增一行来取代。绝不编辑或删除历史。
- 相比手工编写的一次性证据，优先使用由已提交脚本产生的证据（**encode-lessons-in-structure** 原则技能）。

## 对照转录审计日志

运行结束时，在交回之前，检查日志是否如实记录。阅读活动工作区 `agent-transcripts/` 目录下本次运行的转录（系统提示会给出路径）。不要跨 `~/.cursor/projects/*/` 进行通配搜索，那会读取无关的私人聊天。将本次运行的记录与实际发生的事情逐条核对。这些记录的每一段都从本次运行的某个 `start` 行开始（如果本次运行创建了日志，则从第一行开始），并在另一次运行的下一个 `start` 行处结束：

- 检查每一行是否对应一个真实的决策或动作。
- 检查每一行的证据是否可以解析，并且确实展示了该行所声称的内容。
- 如果某个影响了工作走向、却未被记录的分支、转向或被放弃的方案，就是缺口。补充记录。

只更正日志，不更正叙述。审计绝不编辑或删除任何一行，即使是凭空捏造的行。当某行既不是真实的决策也不是真实的动作，或其断言或证据有误时，新增一行来取代它，写明实际发生的事情，并给出可以解析的指针。本次审计不检查本次运行段落之外的行。如果本次运行自身的工作表明其中某行有误，则像处理任何错误决定一样对其进行取代。

## 对轨迹进行跨模型复核

交回之前，派生一个与完成工作的模型不同系列的子智能体。仅自我复核不能替代。该子智能体阅读审计轨迹和本次运行的转录，然后标出用户应当注意的事项。它不是重做工作，而是扫描哪里不够优化或存在风险。按照 **poteto-mode** 技能中"子智能体"一节的规定，为每个子智能体选择其推理等级，以及快速或标准速度。

- 证据薄弱或缺失的决策记录。
- 被跳过的验证步骤，或在转录中未经证明即被声称已完成的验证。
- 事后看来有风险的选择（过早、范围蔓延、掩盖症状）。
- 用户若随意浏览就会漏掉的缺口。

每一份产生了轨迹的运行的回复，都必须以"Attention"一节结尾。先单独一行写出复核者的模型（`reviewed by <model>`），然后列出每条标记，并指向具体的行或时刻。"No flags"是有效的值。模型名称本身不算。

## 复核轨迹

从头到尾阅读，跟随证据指针，抽查。GitHub 会将提交的 TSV 渲染为表格。`column -s$'\t' -t decisions.tsv` 可在终端中渲染它。

## 与其他技能组合

其他技能会将其审计轨迹路由到此处，而不是另起一套。按名称引用本技能，并让它负责格式。不要重复列的定义。
