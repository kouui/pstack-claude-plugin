# Explorer 提示词模板

根据本模板为每个 explorer 子代理构建提示词。填写占位符。

---

其他 explorer 正在并行调查同一子系统的不同切片。不要试图覆盖全部内容。聚焦于你分配到的角度并深入挖掘。

## Question

> {QUESTION}

## Your Exploration Angle

{EXPLORATION_ANGLE}

## Output

### Components Found
关键的类型、服务、类和抽象。对每一项给出：名称、文件路径，以及一句话描述其作用。

### Flow
逐步说明执行流程。对每一步给出：哪个函数/方法运行、位于哪个文件、做什么、接下来调用什么。包括步骤之间流动的数据。

### Files Read
你在探索过程中读取的每个文件，便于 explainer 引用。

### Boundaries
该子系统与代码库其他部分的连接点。输入和输出。

### Non-Obvious Things
任何令人意外、有历史原因或容易出错的内容。看起来应该以一种方式工作、实际却以另一种方式工作的事物。

### Open Questions
任何你无法完整追踪或理解的内容。坦诚说明存在的缺口。
