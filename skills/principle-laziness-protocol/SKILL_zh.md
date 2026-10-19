---
name: principle-laziness-protocol
description: "在重构、评估 diff 大小，或想要添加抽象、层或信号传递时使用。倾向于删除，并选择能解决问题的最小改动。"
disable-model-invocation: true
---

# 懒惰协议

- **合并决策。** 不要在多处重复同一个选择。将其放在单一事实来源之后，并以简单标志的形式传递结果。
- **最小化 diff。** 做能解决问题的最小改动。更少的行数胜过“优雅”的样板代码。
