---
name: typescript-best-practices
description: TypeScript 最佳实践。在读取或编辑任何 .ts 或 .tsx 文件时使用。
paths: ["**/*.ts", "**/*.tsx"]
disable-model-invocation: true
---

# TypeScript 最佳实践

先应用 **type-system-discipline** 原则技能。

| 规则 | 摘要 |
|------|---------|
| 可辨识联合 | 使用 `kind` 字面量判别式为变体建模，使不可能的状态无法被表示。不要使用可选字段袋。 |
| 品牌类型 | 使用 `& { readonly __brand: "X" }` 为原始类型打上品牌，使它们不会被混淆。在边界处验证一次。品牌只能来自经过验证的类型守卫（`input is AgentId`），绝不能来自 `as`。 |
| 构造式建模 | 构造形状，使非法值无法被构造。非空使用 `[T, ...T[]]`，偶数长度使用 `[T, T][]`，范围使用 `start` 加 `duration`。这不是运行时守卫，也不是对细化类型的期望。 |
| 最简全类型 | 只要对其的每个操作都保持全函数性，就保留 `T[]`。仅在宽松类型迫使使用 `!`、强制转换或 "should never happen" 抛错的地方，才强化为 `NonEmpty<T>`。 |
| `unknown` 优于 `any` | 外部数据为 `unknown`。 |
| 先用 schema 再用守卫 | 在手写逐属性的类型守卫之前，使用仓库的运行时 schema 库，并从 schema 推导类型，例如 `z.infer`。 |
| 不使用 `as` 强制转换 | 每个 `as` 都是等待发生的运行时崩溃。只能在验证之后才进行转换。 |
| 收窄层级 | 判别式 switch > `in` 运算符 > `typeof`/`instanceof` > 用户定义的类型守卫 > `as`。 |
| 类型守卫 | 必须验证其所声称的内容。说谎的守卫比 `as` 更糟，因为 bug 隐藏在一个名称表明是安全的东西后面。命名为 `isX` 或 `hasX`。 |
| 穷尽性检查 | 在 default 分支中内联 `const _exhaustive: never = x;`，使新增变体时编译器报错。 |
| `satisfies` 优于 `as` | 在不拓宽字面量类型的情况下验证值。 |
| 边界验证 | 在数据进入的地方解析，转换为命名的领域类型。`Record<string, unknown>`（无论以何种形式书写）止于该解析处。在内部信任类型。参见 **boundary-discipline** 原则技能。 |
| 由 schema 推导类型 | 在声明新接口之前，优先使用 `Pick`/`Omit`/`Parameters`/`ReturnType`/`Awaited`/`typeof`。 |
| 对象参数 | 传递对象而非位置参数，使参数顺序自我说明。在热路径（每帧渲染、分词器、解析器）上可以忽略。 |
| 真实测试 | 能运行的东西不要 mock。优先使用框架真实的测试原语并进行泄漏/可释放检查，并在运行中的构建里验证 UI。只 mock 无法在本地运行的东西。 |
| 结构化遥测 | 优先使用结构化日志诊断，并包含足够的上下文，以便从一个 id 进行调试。发布的代码中不要使用 `console.log`。 |

示例：`references/patterns.md`。
