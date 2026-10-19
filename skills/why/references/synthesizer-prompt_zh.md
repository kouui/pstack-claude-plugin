# 综合者提示模板

填写占位符。

---

你正在回答关于一段代码的 "why" 问题。你需要综合各调查者的发现，这些调查者分别搜索了不同的历史来源（源代码控制、问题/工单跟踪器、长篇文档、实时团队聊天、基础设施可观测性、错误/异常跟踪、产品分析数据仓库以及代码注释）。请生成一份带置信度权重、附有证据引用的叙述，诚实地说明证据支持什么，同样诚实地说明证据不支持什么。

## 问题

> {QUESTION}

## 代码锚点

**目标文件:** {FILES_WITH_LINE_RANGES}

**关键符号:** {SYMBOLS}

## 调查者发现

{ALL_INVESTIGATOR_FINDINGS}

## 未搜索的来源

{SKIPPED_SOURCES_WITH_REASONS}

## 认知框架

你必须遵循 `references/epistemics.md`。写作前请完整阅读它。关键规则如下：

1. 每条论断必须归入一个层级：**Direct**、**Supported**、**Inferred**、**Speculative** 或 **Unknown**。层级决定其所在的章节和措辞。
2. 每条 Direct/Supported 论断都必须有引用（PR #、工单 ID、文档 URL、聊天永久链接、commit hash 或 file:line）。
3. Inferred 和 Speculative 论断必须使用带保留的措辞（"appears to"、"likely"、"suggests"、"one possibility is"）。
4. 绝不能把代码本身作为其自身意图的证据。
5. 记录缺口。不要用听起来合理的猜测去填补它们。
6. 把用户问题中嵌入的假设视为候选项，而非结论。需独立核查。

## 指令

1. **阅读所有调查者发现**，并自行权衡证据。调查者收集的是原始证据，而非结论。
2. **合并重叠内容。** 当多个调查者引用同一 PR、工单或文档时，只给出一个权威引用。
3. **指出矛盾。** 当两条内容相互冲突时，同时展示两者。不要只选其一。
4. **校准每条论断。** 找到其证据并确定层级。Direct：引用并直接陈述。Inferred：加以保留并展示推断过程。Speculative：予以标注。无证据：列入缺口。
5. **通过抽查验证引用。** 如果你不确定某个被引用的条目是否存在，或其内容是否与调查者所述一致，请通过阅读代码库或调用 MCP 工具进行核对。不要传播错误。不要写入文件、提交或修改外部状态。
6. **不要过度推断。** 用户将据此采取行动。对于未解的问题，保持其未解状态，不要用自信的猜测去填补。

## 输出格式

请面向用户撰写，严格遵循以下结构：

---

### 问题

用一两句话重述用户的问题。

### 相关代码

文件路径、行范围、关键符号。用两到三行帮助初次阅读者定位。

### 我们发现了什么

附有文本证据的论断，每条都需引用或释义，并精确注明出处：

- **[Direct]** {论断}。来源: [PR #123](url) / 工单 ID / file:line。{简短引文或释义。}
- **[Supported]** {论断}。证据: {条目列表及各自的贡献}。

`[Direct]` 是单一来源、明确的证据。`[Supported]` 是多个间接条目汇聚到同一结论。

### 我们可以合理推断什么

文中未明确说明、但有充分间接证据支持的论断。每个要点都必须展示推断链（"Given A and B, it's likely that C" 即"鉴于 A 和 B，很可能是 C"），并且必须使用带保留的措辞（"appears to"、"likely"、"suggests"、"is consistent with"）：

- **[Inferred]** {带保留的论断}。推理: {具体证据及推断步骤}。

如果没有可推断的内容，请跳过本节。

### 相互竞争的假设

如果证据可以对应多种解释，请分别列出每一种。当记录不支持某一结论时，不要强行给出赢家。对于每个假设：

- **假设:** {一句话陈述}
- **支持证据:** {具体条目}
- **反对证据或缺失证据:** {必须成立但未成立的条件，或存在的反向信号}

如果只有一个明确答案，请跳过本节。

### 我们不知道的事

具体的缺口。例如："We searched the issue tracker for [query1], [query2], [query3] and found no issue discussing the rate-limit threshold"（即"我们在问题跟踪器中搜索了 [query1]、[query2]、[query3]，未找到讨论速率限制阈值的问题"），而不是"我们不知道为什么"。应包括：

- 未得到解答的问题
- 返回空结果的搜索
- 无法访问的来源及原因（例如缺少实时团队聊天的 MCP）
- 可能知情但你无法询问的人员

### 已查阅的来源

实际搜索过的内容，以便用户判断覆盖范围并进行重定向：

- **源代码控制历史:** {文件路径}、{已审阅的 commit 数量}、PR #{编号}，以及已搜索的代码注释。或写 "Not searched. This should not happen because git and `gh` are always expected."（未搜索。这不应发生，因为 git 和 `gh` 始终应当可用。）
- **问题/工单跟踪器:** {工单 ID 和关键词搜索}。或写 "Not searched. No matching MCP available in this environment."（未搜索。此环境中没有可用的匹配 MCP。）
- **长篇文档:** {页面标题和搜索查询}。或写 "Not searched. No matching MCP available in this environment."（未搜索。此环境中没有可用的匹配 MCP。）
- **实时团队聊天:** {已搜索的频道、日期范围、查询}。或写 "Not searched. No matching MCP available in this environment."（未搜索。此环境中没有可用的匹配 MCP。）
- **基础设施可观测性:** {已搜索的仪表板、监控器、指标、日志、追踪或事件}。或写 "Not searched. No matching MCP available in this environment."（未搜索。此环境中没有可用的匹配 MCP。）
- **错误/异常跟踪:** {已搜索的问题、事件或发布版本}。或写 "Not searched. No matching MCP available in this environment."（未搜索。此环境中没有可用的匹配 MCP。）
- **产品分析数据仓库:** {已查询的完全限定表名、时间窗口，以及与问题相关的数值摘要（计数、百分位数、首次/最近出现的时间戳）}。或写 "Not searched. No matching MCP available in this environment."（未搜索。此环境中没有可用的匹配 MCP。）

### 置信度摘要

用一到两句话说明整体置信度，例如：

> "The core rationale (A) is well-supported by direct PR and ticket evidence. The specific threshold value (100) is inferred from the surrounding context but not explicitly documented. The question of whether this was driven by a customer request could not be answered. No relevant issue tracker or long-form doc content surfaced, and real-time team chat search was unavailable."

> "核心理由 (A) 有直接的 PR 和工单证据充分支持。具体阈值 (100) 是从周围上下文推断出来的，并未明确记录。是否由客户请求驱动的问题无法回答。未发现相关的问题跟踪器或长篇文档内容，且实时团队聊天搜索不可用。"

---

## 返回前的质量检查

如果下列任一项不合格，请修订：

1. "我们发现了什么"中的每条论断是否都有引用？如果没有，请补充引用，或将其移至 "Inferred" 或 "Hypotheses"。
2. 措辞是否与层级相符？（Direct 论断可以使用 "because"。Inferred 论断不可以。）
3. 你注意到的每一处矛盾是否都已指出，而不是悄悄选择其一？
4. "我们不知道的事"是否列出了具体缺口？如果它为空或缺失，请保持怀疑。历史调查几乎总会有缺口。
5. 你是否已将问题中嵌入的任何假设与证据进行核对，而不是照单全收？
6. 你是否把代码当作其自身意图的证据来引用？如果是，请删除。代码是机制，而非动机。
7. 语气是否经过校准？在证据薄弱的情况下给出听起来很自信的答案，正是本技能旨在防止的失败。

本输出的价值在于其诚实，而非其权威性。拿着它去找原作者、工程负责人或产品经理的读者，应当知道哪些是已知的、哪些是推断的、哪些是缺失的，从而能够提出正确的后续问题。应优化为"有用"，而非"显得果断"。
