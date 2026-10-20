# 在修改代码之前先理解它

不理解的代码就去修改，是细微回归被发布出去的原因，对 agent 也是如此。agent 通常以两种方式失败：要么误解了你想要什么，要么缺少把工作做好所需的上下文。[prompt 里写什么](./02-poteto-mode.md#what-goes-in-a-prompt) 解决的是第一种情况，本页解决的是第二种。

pstack 提供四种入口。`/how` 解释代码现在做了什么。`/why` 挖掘它为何会是这种结构的原因。`/teach` 把两者融合成一个解释。`/recall` 重建你自己关于某个主题的近期上下文。每一个都会让 agent 用你可以核对的文字解释自己。这就是你监督一个可能比你更了解代码的 agent 的方式。

![一位侦探拿着放大镜研究机器蓝图，机器人在搬运案卷；她身后的证据板上，用线把 /how 和 /why 下的线索连在一起。](../images/understanding.jpg)

## 从只读调查开始

当原因不清楚时，要结论，不要修复：

```text
/poteto-mode investigate why background jobs time out every few hours. give me what we know, what data you used, and your best hypotheses. don't change any code yet.
```

“don't change any code yet” 会把它路由到 [调查手册](../../../skills/poteto-mode/playbooks/investigation.md)。该手册运行 `/how`，对于动机相关的问题再加上 `/why`，并返回带引用的解释。对于在多个选项之间做选择的情况，它返回一个附带权衡表的建议。询问“what data you used”会让 agent 把证据与猜测区分开。当调查结果指向某个修复时，请把修复作为一个新任务开始。

## 用 `/how` 追踪行为

```text
/how do we dedupe notifications? is there an n+1 when we look up subscribers?
```

问你真正想问的问题。[`/how`](../../../skills/how/SKILL.md) 会读取代码，并以资深工程师带你熟悉子系统的水平进行回答，包括运行时流程、关键类型以及不显眼的部分。对于大型子系统，它会先派出两到四个只读的探索者。对于范围较窄的问题，它只是阅读并解释。

## 用 `/why` 挖掘历史

```text
/why was the retry limit set to five? does the reason still hold?
```

[`/why`](../../../skills/why/SKILL.md) 从源代码管理开始，然后并行查询你的 MCP 所暴露的各类证据，例如 issue 跟踪器、长篇文档、团队聊天、可观测性、错误追踪和分析工具。报告会引用所有内容，区分直接证据与推断，并且在记录稀少时使用“appears to”（看起来）。空结果也会被报告，因为“没有人记录过原因”本身就是一个答案。

两者很自然地可以组合使用。当你怀疑历史能解释当前的混乱时，`do why first then how` 是一个很好的 prompt。

## 用 `/teach` 真正理解

```text
/teach me how this PR changes retries. convince me it fixes the cause and not the symptom.
```

[`/teach`](../../../skills/teach/SKILL.md) 适用于摘要不够用的时候。它会运行 `/how` 和 `/why`（对于小改动，可能只运行其中一个），并把发现编织成一段平实的解释，逐张图示层层展开。“convince me”这种写法值得反复使用。它把解释变成一个你可以检验的论证，而不是一次走马观花的导览。

它也可以用于 agent 自己的选择：

```text
/teach me why you implemented it this way and not with a queue. what did you trade off, and why?
```

教学对 agent 的帮助和对你的帮助一样大。一个必须解释自己工作的 agent，必须阅读代码，并为每个论断给出证据，而不是自信地陈述后就继续前进。

## 用 `/recall` 重建你自己的上下文

```text
/recall catch me up on the export work from last week
```

[`/recall`](../../../skills/recall/SKILL.md) 会挖掘你自己近期的对话，以及共享记录（issue、先前的修复、仍在触发的错误），并返回一份关于当前进展和下一步的简报。你以前的对话中保存着新 agent 缺少的上下文，所以在旧主题上开始新工作时，先加载它，再给出新的输入：

```text
/recall my work on the virtualized list from yesterday, then read this bug report.
```

如果你想恢复某一次特定的对话，那应该使用下面的会话接手手册，而不是 `/recall`。

## 用会话接手接管先前的工作

当另一个 agent（或者上周的你）留下了一个做到一半的分支时：

```text
/poteto-mode take over this branch. read the decision log, figure out what's done, and continue from there. don't redo finished work.
```

[会话接手手册](../../../skills/poteto-mode/playbooks/session-pickup.md) 把先前的记录视为权威。它重建分支状态和决定，指出恢复的节点，并根据原始目标验证继承下来的结论，而不是从头重新推导一切。

**常见误区：** 不要因为“agent 反正会自己读代码”就跳过本页介绍的 skill。一个在没有追踪过模型的情况下就开始编辑的 agent，往往会在第一个看似合理的位置修复症状。先用 `/how`，比第二次修 bug 更省力。

下一步：[设计改动](./04-design.md)。
