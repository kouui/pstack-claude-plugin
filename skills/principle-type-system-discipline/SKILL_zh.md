---
name: principle-type-system-discipline
description: "在设计类型、审查函数签名或用任何静态类型语言编写代码时应用。让非法状态无法表示，为语义原语打上品牌类型，在边界处解析外部数据，穷举变体，并从权威 schema 派生类型。"
disable-model-invocation: true
---

# 类型系统纪律

让编译器拒绝不可能的状态、不匹配的原语以及未处理的变体。优先通过定义让错误和特殊情况不存在，而不是不断增加处理函数。TypeScript 语法参见 `typescript-best-practices` skill。

- **为语义原语打品牌。** `UserId` 和 `OrderId` 底层都是字符串，但不应可以互换。为每一个分别定义类型，例如 TypeScript 中的品牌交叉类型，其他语言中则使用 newtype。在创建时校验一次，下游信任该类型。如果两个参数共享同一原语类型但含义不同，就为它们打上品牌。
- **外部数据在解析前是无类型的。** 包括 RPC 载荷、JSON、IPC 消息、CLI 参数、配置文件、环境变量、数据库行。在每个边界处设置解析函数，把非结构化输入转换为有类型的模型。关于校验应放在何处，参见 **boundary-discipline** principle skill。
- **从权威 schema 派生类型。** 当 protocol buffer、OpenAPI 规范、GraphQL schema、数据库迁移、设计 token 文件或其他文件负责定义某个形状时，应从该文件派生类型，而不是手写一份平行的类型。参见 **encode-lessons-in-structure** principle skill。
