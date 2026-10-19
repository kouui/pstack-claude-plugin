# 用原则名称来引导

pstack 把 24 条原则作为独立的 skill 提供。`/poteto-mode` 在每个多步骤任务开始时读取它们的索引，应用任务触发的那些原则，并在回复中点明每条被应用的原则以及它改变了哪个决定。

你不需要调用原则。你用它们的名字来引导 agent。每个名字都指向一条 agent 已经读过的完整规则，所以一句话就能比一大段指令更精确地调整工作方向。

## 实际引导

假设 agent 准备在三个现有适配器旁边再加一个新适配器：

```text
use subtract before you add. delete the obsolete adapters first, then design what's left.
```

假设 agent 因为构建通过就宣称成功：

```text
apply prove it works. run the real import flow and show me the written records.
```

假设两个并行的尝试即将写入同一个分支：

```text
separate before serializing shared state. give each attempt its own worktree, no locks.
```

每一句话都能生效，是因为背后的规则够具体。agent 仍然需要在回复中说明，它的哪个决定被这条规则改变了。只有引用而没有对应决定，说明 agent 只是说出了规则的名字，并没有真正应用它。

## 24 条原则简述

核心原则决定要构建多少，以及何时重新考虑设计：

- [Laziness Protocol（懒惰协议）](../../skills/principle-laziness-protocol/SKILL.md) 优先删除，以及用最小的改动解决问题。
- [Foundational Thinking（基础性思考）](../../skills/principle-foundational-thinking/SKILL.md) 在编写逻辑之前先选定核心数据结构。
- [Redesign from First Principles（从第一性原理重新设计）](../../skills/principle-redesign-from-first-principles/SKILL.md) 把新需求整合进去，就像它从第一天起就存在一样。
- [Attack the Premise（攻击前提）](../../skills/principle-attack-the-premise/SKILL.md) 写下两次或更多失败修复共同依赖的前提，并在下一次修复之前直接对其进行检验。
- [Subtract Before You Add（先减后加）](../../skills/principle-subtract-before-you-add/SKILL.md) 在其之上构建之前，先移除无用的部分。
- [Minimize Reader Load（降低阅读负担）](../../skills/principle-minimize-reader-load/SKILL.md) 合并读者必须在脑中记住的层级和隐藏状态。
- [Outcome-Oriented Execution（面向结果的执行）](../../skills/principle-outcome-oriented-execution/SKILL.md) 让重写朝目标设计收敛，而不是保留临时的兼容状态。
- [Experience First（体验优先）](../../skills/principle-experience-first/SKILL.md) 选择用户得到的结果，而不是实现上的便利。
- [Exhaust the Design Space（穷尽设计空间）](../../skills/principle-exhaust-the-design-space/SKILL.md) 在没有先例可循时，构建两到三个相互竞争的原型。
- [Build the Lever（构建杠杆）](../../skills/principle-build-the-lever/SKILL.md) 构建能完成或证明这项工作的脚本，使审阅者可以重新运行。当 agent 反复手动做同一件事时，让它编写自己希望拥有的工具或 skill。如果某个步骤每次都能以同样的方式由脚本完成，就用脚本，把 agent 留给需要判断的环节。

架构原则决定状态、校验和兼容性放在哪里：

- [Model the Domain（建模领域）](../../skills/principle-model-the-domain/SKILL.md) 把重复出现的规则编码进同一个结构，而不是分散在各处的条件判断中。
- [Boundary Discipline（边界纪律）](../../skills/principle-boundary-discipline/SKILL.md) 在边界处校验，并信任内部类型。
- [Type System Discipline（类型系统纪律）](../../skills/principle-type-system-discipline/SKILL.md) 让非法状态无法被表示。
- [Make Operations Idempotent（使操作幂等）](../../skills/principle-make-operations-idempotent/SKILL.md) 让重试收敛到同一个最终状态。
- [Migrate Callers Then Delete Legacy APIs（迁移调用方再删除旧 API）](../../skills/principle-migrate-callers-then-delete-legacy-apis/SKILL.md) 在同一波次中完成迁移和删除。
- [Separate Before Serializing Shared State（先分离再串行化共享状态）](../../skills/principle-separate-before-serializing-shared-state/SKILL.md) 在增加协调机制之前，先移除共享。

验证原则定义什么才算证明：

- [Prove It Works（证明它能工作）](../../skills/principle-prove-it-works/SKILL.md) 验证真实的产物，而不是替代品。
- [Fix Root Causes（修复根本原因）](../../skills/principle-fix-root-causes/SKILL.md) 在修改代码之前先复现问题并追溯到原因。
- [Sequence Work into Verifiable Units（把工作拆成可验证的单元）](../../skills/principle-sequence-verifiable-units/SKILL.md) 让每个小单元以一次检查结束，然后再开始下一个。
- [Test Behavior, Not Implementation（测试行为，而不是实现）](../../skills/principle-test-behavior-not-implementation/SKILL.md) 断言代码的用户能观察到的行为，使行为被破坏时测试会失败，并且在内部实现变化时测试依然成立。
- [Explain the Number（解释这个数字）](../../skills/principle-explain-the-number/SKILL.md) 在任何人信任或汇报一个测量数字之前，先说明限制它的因素，并排除它测的其实是别的东西的可能。[`/benchmark-checklist`](../../skills/benchmark-checklist/SKILL.md) 把它变成七个问题，由你根据真实运行结果来回答。

委派原则让并行工作保持有序：

- [Guard the Context Window（守护上下文窗口）](../../skills/principle-guard-the-context-window/SKILL.md) 把大量阅读工作交给子 agent，并把发现保留在主对话中。
- [Never Block on the Human（不要被人工阻塞）](../../skills/principle-never-block-on-the-human/SKILL.md) 对可逆的工作继续推进，并呈现结果。

还有一条元原则：

- [Encode Lessons in Structure（把经验编码进结构）](../../skills/principle-encode-lessons-in-structure/SKILL.md) 把你重复过两次的建议变成 lint、检查或脚本。[`/correct`](../../skills/correct/SKILL.md) 把它应用到整个仓库，如 [让它成为你的](./09-make-it-yours.md#fix-the-environment-with-correct) 中所示。

不必死记这份列表。现在先浏览一遍，等你发现 agent 做了某件本可以由这里的某个名字避免的事情时，再回来看。这样词汇才会记住。

下一步：[让它成为你的](./09-make-it-yours.md)。
