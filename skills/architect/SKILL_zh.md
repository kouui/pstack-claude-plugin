---
name: architect
description: "在编写代码之前先草拟类型、签名和模块结构,然后在实现填充过程中持续参与。用于 /architect、'architect this'、'design this',或在直接写代码会锁定错误形态的非平凡工作。"
disable-model-invocation: true
---

# Architect

在 Claude Code 中,每个会话开始时先阅读一次 [Claude Code 适配器](../../docs/harness/claude-code.md),然后再按本 skill 操作。它把 pstack skill 中的 Cursor 名称映射到 Claude Code。

用 `not implemented` 函数体和伪代码草拟类型、函数签名、类的形状以及模块边界。综合多个模型视角的结果,然后依据选定的草图填充代码。

## 开始

开始前,为每个阶段打开一个待办列表,每个阶段一项。

1. Ground(夯实基础)
2. Sketch(草图)
3. Agree(达成一致)
4. Implement(实现)
5. Scrap(废弃)

## 阶段 A:夯实问题基础

为新代码涉及的每个系统建立真实的心智模型。对相关子系统运行 **how** skill。只说出文件名不算夯实基础。产出 `how` 所规定的架构说明。如果设计重新定义了所有权或分层,还要对现有结构运行 **why** skill,使其理由成为约束,而不是猜测。

只有当工作确实是全新开发、没有周边系统需要集成时,才可以跳过阶段 A。

## 阶段 B:草图

使用设计草图任务和阶段 A 的基础产物运行 **arena** skill。将 `references/runner-prompt.md` 作为每个 runner 的 prompt 传入。

runner 取自 `pstack-models.md` 规则中的 `architect runners` 行,而不是 `arena runners` 行。如果规则或该行缺失,使用 `opus high` 和 `sonnet high`。别名条目和被拒绝的条目遵循 **arena** skill 阶段 A 中的 runner 规则。

设计要想两遍。即使第一个方案看起来已经足够,在综合之前也要求至少两个结构上不同的候选方案。这是 **exhaust-the-design-space** 原则 skill 的具体化。应比较整体形状的替代方案,而不是在同一形状内做局部修补。

在综合之前,依据 [`references/design-red-flags.md`](references/design-red-flags.md) 筛查每个候选方案。假设下一个贡献者是一个只能看到它打开过的文件的 agent,它会复制最近的示例,并走编译通过的最短路径。优先选择这样的设计:从某一个文件看起来正确的改动,对整个仓库也是正确的。

比较可行候选方案时关注接口深度。优先选择能把更多复杂性隐藏在更小、更简单的公共接口之后的设计。丰富的接口可以通过集中能力而不是将其分散在各层中,从而缩短调用链。

Arena 返回一个综合后的设计包。其综合决策填入理由文档的"Synthesis decision"部分。

## 阶段 C:达成一致(可选)

默认情况下:直接以综合后的设计进入实现阶段。不设人工检查点。

当调用者明确要求时,可以选择加入检查点,例如"/architect with checkpoint"、"stop and show me before implementing" 或类似表述。然后展示综合后的设计并暂停,等待确认。

无论哪种方式,综合结果都可以作为独立提交,即 **foundational-thinking** 原则 skill 的"scaffold first"模式。根据 **outcome-oriented-execution** 原则 skill,填充过程中有计划、有范围的破坏是可以接受的。若要在实现前对设计施加对抗性压力,请对综合后的草图运行 **interrogate** skill。

如果人类对形状提出异议(在检查点中或事后),将其视为阶段 A 的证据。在编写更多代码之前,重新夯实基础并重新运行阶段 B。

## 阶段 D:依据草图实现

用代码替换 `not implemented` 函数体,用逻辑替换伪代码。综合后的草图是契约。与之偏离的地方是需要提出的信号,而不是默默吸收的摩擦。如果某个函数需要草图未预料到的参数,请询问是草图错了、需求遗漏了,还是实现越界了。

## 阶段 E:当架构错误时废弃

如果实现过程中持续出现草图无法消化的摩擦,就把草图扔掉。依据 **redesign-from-first-principles** 和 **fix-root-causes** 原则 skill,不要把修补硬加到错误的设计上。

信号是一种*模式*,而不是单个实例。迹象包括:

- 相同形态的变通方案在互不相关的代码中反复出现。
- 多个互不相关的边界情况都需要特殊分支。
- 类型需要逃生口(`any`、类型转换、实践中总是被设置的可选字段)才能编译。
- 当草图说状态不是共享的时,却出现"我们需要一把锁"的条件反射。
- 调用者必须了解抽象的内部规则才能使用它。
- 实现中出现两个或更多、同一形态的独立阶段 D 偏离。

请依据判断。少数边界情况不足以否定一个架构。有些问题本质上就是复杂的。数据的复杂性不等于设计的复杂性。

废弃时:

1. 对已构建的内容重新运行 **how** skill。
2. 按 redesign-from-first-principles,把新约束当作第一天就有的假设来重新设计。
3. 依据 **subtract-before-you-add** 原则 skill,先删减再添加。新草图在增长之前应当比旧草图更小。
4. 返回阶段 B,重新运行 arena。

## 输出

调用者的用法先写出,再由其推导出类型草图。对于小改动,输出一个包含新类型和签名的文件。对于较大的工作,输出模块映射加类型定义。理由文档随之提供,其格式遵循 `references/rationale-template.md`,包括用法草图和综合决策。
