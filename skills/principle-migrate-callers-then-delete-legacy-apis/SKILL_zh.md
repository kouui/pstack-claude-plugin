---
name: principle-migrate-callers-then-delete-legacy-apis
description: "在引入新的内部 API 而旧调用方仍然存在时使用。在同一波次中迁移调用方并删除旧 API，而不是保留兼容层。"
disable-model-invocation: true
---

# 迁移调用方，然后删除旧 API

- 更新测试以断言新契约，并删除那些只保护重构前实现细节的测试

**适用场景：**
- 没有外部用户依赖向后兼容性
- 项目能够承受协调好的破坏性变更
- 新 API 是简化或重构计划的一部分
