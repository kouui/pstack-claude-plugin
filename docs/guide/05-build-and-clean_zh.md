# 构建改动并清理 diff

各个构建手册遵循同一个原则：说明你观察到了什么，并让手册来要求证据。本页展示每个常见构建任务的 prompt 应该怎么写，然后介绍让 diff 保持可审阅的清理习惯。

## 用你已知的信息写每个构建手册的 prompt

Bug 类 prompt 说明症状，并要求先复现：

```text
/poteto-mode this command emits two records after a retry. repro first, then fix and verify.
```

功能类 prompt 说明行为，以及什么不能改变：

```text
/poteto-mode add a --json flag. text output stays byte-identical. verify both forms.
```

重构类 prompt 在结构移动之前固定行为：

```text
/poteto-mode move parsing into one module, zero behavior change. record the current output first and prove it's unchanged after.
```

性能类 prompt 说明测量值，而不是印象：

```text
/poteto-mode startup takes 1.8s on this fixture. trace it, fix the measured cause, show me before and after.
```

这些都会路由到对应的手册（[Bug 修复](../../skills/poteto-mode/playbooks/bug-fix.md)、[功能](../../skills/poteto-mode/playbooks/feature.md)、[重构](../../skills/poteto-mode/playbooks/refactoring.md)、[性能问题](../../skills/poteto-mode/playbooks/perf-issue.md)），手册会补上你没有输入的步骤：修复前先复现，实现前先说明数据形态，重构前先固定行为，优化前先做性能分析。

如果要持续改进某一个数字，可以使用 [Hillclimb 手册](../../skills/poteto-mode/playbooks/hillclimb.md)。给它指标、目标以及尝试次数的下限，它会在一个冻结的测量工具下，每次只验证一个假设。它保留有效的改进，并撤销其他所有改动。

两个性能手册都会对它们的数字运行 [`/benchmark-checklist`](../../skills/benchmark-checklist/SKILL.md)。性能问题手册会审查基线以及之后的每一个数字，Hillclimb 则会在冻结测量工具之前审查它。[验证与交付](./06-verify-and-ship.md#vet-a-measured-number-with-benchmark-checklist) 展示了何时需要你自己输入这条命令。

有时你想在修复之前先找到原因。对于正在发生的现象，例如内存泄漏、空闲时 CPU 占满或视觉故障，[运行时取证手册](../../skills/poteto-mode/playbooks/runtime-forensics.md) 会对正在运行的进程进行插桩。对于你已经采集好的性能分析文件，[Trace 取证手册](../../skills/poteto-mode/playbooks/trace-forensics.md) 会读取该产物，并把热点帧映射到源代码。两者返回的都是诊断，而不是修复：

```text
/poteto-mode here's a cpuprofile from the slow startup. tell me where the time goes and which source lines own it. no fix yet.
```

## 用 `/tdd` 先写失败的测试

当一个 bug 有成本较低的本地测试路径时，整个 prompt 可以只有两个词：

```text
/tdd implement
```

在上下文中，这就足够了。[`/tdd`](../../skills/tdd/SKILL.md) 会先写出因预期原因而失败的最小测试，然后进行修复，并重新运行该测试。如果某个测试需要大量的 harness 搭建或脆弱的 mock，skill 会说明这一点，并改用最接近的可执行检查。不要为了测试而硬塞测试；如果真实命令能提供更有力的证据，就用真实命令。

## 按名称加载 TypeScript 规则

[`typescript-best-practices`](../../skills/typescript-best-practices/SKILL.md) 把类型系统原则转化为具体规则：判别联合、边界处使用 `unknown`、穷尽的变体、由 schema 推导类型。它不会自动加载，因此当任务涉及 `.ts` 或 `.tsx` 文件时，请输入 `/typescript-best-practices`。

## 提交前先清理

[提交 PR 手册](../../skills/poteto-mode/playbooks/opening-a-pr.md) 在每次 commit 之前对 diff 运行 `/deslop`，并对 PR 描述和 commit 正文应用 [`/unslop`](../../skills/unslop/SKILL.md)。`/deslop` 随 `cursor-team-kit` 插件提供，而不在 pstack 中。如果你没有安装它，就用普通的话直接提出同样的要求：删除叙述性注释、没有依据的防御性判断、无用的兼容路径，以及无关的改动。

对于文字内容，`/unslop` 接受一个目标，以及你自己的额外规则：

```text
/unslop the readme changes, no emdashes
```

你会形成自己的简写习惯。即使是像 `unslop that, tighten it` 这样简短的 prompt，这个 skill 也能正确理解意图。

## 用 `/no-comments` 清理注释

注释需要单独的一轮检查，而且不应该由写下它们的 agent 来做。作者会像你为自己的代码辩护那样为自己的注释辩护。因此在审阅之前，把注释交给全新的视角：

```text
/no-comments the diff
```

[`/no-comments`](../../skills/no-comments/SKILL.md) 会派生 [Comment Sicko](../../agents/comment-sicko.md)，这是一个只读的审阅者，保留清单很短：许可证头、公共 API 的文档注释、解释代码无法表达之内容的链接，以及外部依赖强制要求的、你无法重构掉的行为。其他所有注释都会被删除。你自己代码中的意外之处不会获得这种豁免。审阅者会把这条注释作为重构标记返回，`/no-comments` 会从根本原因上修复它接受的标记。当注释声明了一个约束，例如“不要删除”，这个 skill 会提议把该声明编码为类型、测试或 lint。无论哪种情况，这条注释都会被删除。

分工值得分清楚。`/deslop` 清理代码中的废话，`/unslop` 清理文字中的废话，而 `/no-comments` 把注释交给一位没有写过它们的审阅者。

**常见误区：** 清理不是可有可无的打磨。一个带有叙述性注释和防御性无用代码的 diff，会让审阅者觉得没有完成，而多余的代码正是下一个 bug 藏身的地方。如果 diff 感觉臃肿，就在提交之前说 `deslop it`，而不是等审阅者指出来。

下一步：[验证与交付](./06-verify-and-ship.md)。
