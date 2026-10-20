# 验证结果并开 PR

"能编译" 不是证据。[Prove It Works 原则](../../skills/principle-prove-it-works/SKILL.md) 要求 agent 在报告成功之前检查真实产物，而你的工作是让 "真实产物" 可以被核对。本页讲的是：说明完成条件、审查一个测量数字、为你的应用生成验证技能、用形式化方法验证测试很少覆盖到的属性、开 PR，以及把 PR 推进到合并。

验证通常是大多数 agent 工作中最慢的一步，因为它往往要等人来做。让 agent 能自己完成验证，你就不再是瓶颈。跳过验证，多开几个 agent 只会产生更多没人检查的工作，等着你去审。

![A prototype plane flies a real test course while she times it with a stopwatch and robots film and checklist the run; the terminal reads verify: pass, evidence: captured.](./images/verification.jpg)

## 一开始就说明完成条件

在第一条提示词里说明 "完成" 指什么，措辞随你怎么顺手都行：

```text
/poteto-mode add json output to this command. text output stays byte-identical, the json parses, both run against the sample project. show me the evidence.
```

这样 agent 就有了三项它可以去执行的检查，而不是一种需要它去迎合的感觉。回复返回时，应当附上确切的命令和输出。如果某项检查无法运行，一个好的回复会说 "inconclusive"（无定论）。没有证据却很有把握的回复，是一个警示信号。

让检查方式与改动匹配：

- CLI 改动运行真实命令。
- UI 改动在运行中的应用里走一遍改动的流程。当它必须与参考图逐像素一致时，[视觉一致性操作手册](../../skills/poteto-mode/playbooks/visual-parity.md) 会把截图与冻结的基线做对比，而不是凭肉眼判断。
- 解析器或迁移改动重放一份已保存的输入。
- 性能改动比较改动前后的 profile。
- 存储改动读回写入的值。

要求以你自己可以检查的产物形式提交证明：先失败、再通过的测试，改动前后的视频，trace，截图。如果修复已经合并，就要求在 main 上再做一次同样的检查。产物胜过一个听起来合理的解释，因为你可以在不重跑整个过程的情况下质疑它。

对于你并不完全信任的小 diff，[`/blast-radius`](../../skills/blast-radius/SKILL.md) 会找出它可能在别处造成的破坏。它找到那个让改动安全的关键事实，并通过运行代码来证明这一事实，而不是靠争辩。

## 用 `/benchmark-checklist` 审查测量数字

前后对比的数字是最容易不小心算错的证据。热缓存、一侧是 debug 构建，或者有工作没有在计时区间内运行，都可能产生一个令人信服的加速比。在你报告或据此行动之前，输入：

```text
/benchmark-checklist vet the export speedup before it goes in the pr
```

[`/benchmark-checklist`](../../skills/benchmark-checklist/SKILL.md) 会问七个问题，并要求每个问题都有一次运行的证据：

1. 什么限制了这个数字，为什么它不是翻倍？
2. 每一侧运行时，是否都按照生产环境的方式调优？
3. 结果是否突破了物理极限，比如磁盘带宽或核心数？
4. 是否有任何东西报错或返回了错误输出？
5. 在交替运行中能否复现，并给出中位数和范围？
6. 在用户等待的路径上，它端到端是否真的有意义？
7. 工作是否确实发生在计时区间之内？

结论会是 faster（更快）、slower（更慢）、no measurable difference（没有可测量的差异）或 inconclusive（无定论）之一，并附上运行次数、范围和限制因素。如果它无法指出限制因素，或者某一侧运行时没有调优，结论就是 inconclusive。`/poteto-mode` 已经在 Perf issue 和 Hillclimb 操作手册里面运行了这份清单，所以只有当你在这些操作手册之外做了测量，或者别人的数字看起来好得不太真实时，才需要你自己输入。它是[解释数字原则](../../skills/principle-explain-the-number/SKILL.md) 的可执行形式。

## 创建项目验证技能

上面 UI 那一条隐藏了一个真实的要求：agent 需要一种脚本化的方式来驱动你的应用。如果你的项目已经有，那很好。如果没有，运行：

```text
/create-verification-skill
```

[`/create-verification-skill`](../../skills/create-verification-skill/SKILL.md) 访谈的是仓库，而不是你。它会弄清楚用户会接触什么、应用如何在本地启动、什么可以驱动它（先看现有的 harness，否则是浏览器与 CDP、PTY，或普通 HTTP），什么证据能证明行为正确，以及两个实例能否并行运行。它只会问代码无法回答的问题。

它会写出 `.claude/skills/verify-<app>/`，其中包含面向 agent 的说明，带有确切的 Launch、Doctor、Drive、Evidence 和 Cleanup 小节，为它附带的脚本提供 Helpers 小节，并在 `features/` 下提供功能映射，索引应用做什么以及哪种结果能证明每项功能正常。该技能附带了一个[功能映射示例](../../skills/create-verification-skill/references/feature-map-example/)，包含 README 索引，以及每项功能一个文件，并使用四个必需的二级标题。交付之前，生成器会端到端地证明一次该技能：启动、doctor 检查、驱动一项功能、采集证据、清理，然后确认证据在清理之后仍然保留。如果这一证明失败，就不要使用输出。

从此以后，"在应用里验证它" 就成了任何 agent 都能在这个仓库中执行的一个步骤，无需任何设置对话。当你希望以特定形式拿到证明时，在提示词中指名：

```text
/poteto-mode build the bulk-archive action. use /verify-<app> to verify your changes and show me a video and screenshots as proof.
```

```text
/poteto-mode repro this with /verify-<app>. if it repros on main, fix it and show me a video as proof.
```

验证技能跑通之后，[`/swarm`](../../skills/swarm/SKILL.md) 可以按功能映射的条目拆分一整轮检查并汇总结果。一群验证者在样本足够大时也能确认性能提升，或者在 PR 发布前对应用做回归模糊测试。

把验证技能当作基础设施，而不是一次性的东西。把它提交进仓库，让团队里每个人和每个 agent 都以同样的方式驱动应用。然后[构建杠杆](../../skills/principle-build-the-lever/SKILL.md)。当 agent 反复写一次性脚本去点击应用时，就要求它改为调用一个小的控制 CLI，由验证技能来调用。这样 agent 消耗的 token 更少，每次运行也都可以复现。一个 agent 用得好的 CLI 通常具备以下特点：

- 几个可组合的命令，每个都做实际的工作，而不是很多只做一点点的细碎命令。
- 对任何有破坏性的操作提供 `--dry-run` 选项。
- 子命令逐步展示功能，而不是一次全部展示。
- 错误信息会告诉你该怎么做。
- 丰富的 `--help` 文本。
- 机器可读的输出，比如 JSON。

在这个过程中，也把开发环境的搭建变成可重复的：预置数据、测试用户，以及一条每次都以相同方式把环境启动起来的命令。

## 保持验证技能诚实

应用会变，功能映射也会过时。请至少每天运行一次，最好由定时自动化任务触发，这样就不需要有人去记：

```text
/maintain-verification-skill
```

[`/maintain-verification-skill`](../../skills/maintain-verification-skill/SKILL.md) 审计生成的技能：为每项功能并行运行一个只读的源码阅读器，然后执行一轮实时检查，驱动每一个已映射的功能。它的结果只能是以下三种之一：`clean` 表示覆盖完整、无需发布任何内容；`changed` 表示发布一个 PR，内含经过证明的修正，且改动仅限于验证技能自己的目录；`blocked` 会指出阻碍原因。它从不修改产品代码。如果实时检查发现了产品回归，它会报告该回归，而不是用文档修改把它掩盖掉。

## 用形式化验证技能证明一个属性

验证技能只能证明你驱动过的路径。有些 bug 藏在没有人去驱动的路径上：两次重试重叠、锁在写入中途过期、状态变化乱序到达。测试很少能覆盖到这些情况。对于此类 bug 代价高昂的代码，运行：

```text
/create-formal-verification-skill
```

[`/create-formal-verification-skill`](../../skills/create-formal-verification-skill/SKILL.md) 会从代码中挑选目标：即代码所做的某个承诺，一旦被打破会伤害用户，而测试很少能触及那个打破它的情况。它挑选的是最强的目标，而不是最容易验证的目标。如果你想从某个领域或某个 bug 开始，可以直接点名：

```text
/create-formal-verification-skill the job lease and retry logic. a job must never run twice at once.
```

问题决定使用什么工具。事件是否可能以错误的顺序发生，适合用模型检查器作用于设计的一个小模型上，而 TLA+ 是任何语言下这类问题的默认选择。代码对每个输入是否都正确，适合用直接作用于代码的验证器。如果你的语言没有可用的验证器，agent 会明说，并用生成的输入来测试真实代码，而不是用手写的函数模型。一个必须在任意规模下都成立的属性，可能需要一个证明。agent 会告诉你它选了哪种工具以及为什么。

它只验证一个属性，并且会在一个被故意破坏的代码变体上展示该属性失败，因为一个你从未见过它失败的检查，什么也证明不了。如果当前代码已经打破了该属性，你会得到一个反例，用你代码术语写成的简短步骤序列。该属性保持原样陈述，因此这个发现仍然只是一个发现。除非你要求，否则产品代码不会改动。

它还会写一个项目内的技能，告诉下一个 agent 某次改动是否触及已验证的代码、如何用一条命令运行该检查，以及如何区分真正的 bug 和建模错误。交付之前，生成器会在一个打破该属性的临时改动上按照该技能走一遍，而检查必须能抓住它。交付说明会写明验证了什么、使用了什么工具和边界、检查发现了什么，以及没有覆盖什么。一次通过只覆盖模型或边界，而不是整个系统。

该技能还会记录每项检查覆盖哪些代码，以及检查最后一次与之匹配的提交。合并和重构如果绕过了该技能，检查可能仍然在一份已经不存在的代码描述上通过。之后，请运行 [`/maintain-formal-verification-skill`](../../skills/maintain-formal-verification-skill/SKILL.md)。它会根据代码更新验证，重新运行，证明每个更新后的检查仍然能够失败，并告诉你每个失败究竟是验证本身的漂移，还是代码中的 bug。

## 开 PR

```text
/poteto-mode open the pr. small ordered commits, evidence in the description.
```

[开 PR 操作手册](../../skills/poteto-mode/playbooks/opening-a-pr.md) 在 worktree 中工作，把改动 rebase 成小而有序的提交，清理 diff，润色文字，然后返回 PR 链接。五个窄的 PR 胜过一个大而全的 PR，堆叠的后续 PR 也胜过一个不断变大的分支。

## 用 Babysit 把 PR 推进到可合并状态

一个已开的 PR 会立即开始累积阻碍：检查失败、评审者留言、主干前移。把这些琐碎的变动交给 [Babysit 操作手册](../../skills/poteto-mode/playbooks/babysit.md)：

```text
/poteto-mode babysit this pr. get it green.
```

Babysit 会监视 PR，在可用时借助内置的 GitHub 或 Origin CLI 监视器，并按顺序处理阻碍：先冲突，再评审线程，然后是 CI。每个已知修复都会合并成一次推送，因此检查只会重启一次，而不是每次修复后都重启。评论分拣是持怀疑态度的，因为人类和机器人会在同一个列表里提交真实的发现和噪音。真实的发现会得到修复，噪音则会被驳回，并在对应线程上贴出反驳证据。如果你只想要状态，那就问得更简单一些，Babysit 会直接回答，而不会启动循环：

```text
/poteto-mode check on pr 123. anything outstanding?
```

Babysit 止步于可合并状态。它从不合并，即使一切都是绿色的，因为合并是另一个决定。

## 用 Shipping 落地整个堆栈

绿色并不等于安全。当你准备落地时，直接说明：

```text
/poteto-mode land the stack.
```

[发布操作手册](../../skills/poteto-mode/playbooks/shipping.md) 会在任何东西被挂上自动合并之前，独立验证每个 PR。每个 PR 由一个全新的 agent 实时证明其行为，而判断一项改动的 agent 永远不是写它的那个 agent。然后，Shipping 只会从底部开始，落地那段连续且已验证的部分，每次一个 PR，默认通过 GitHub 完成，若 Origin 的 CLI 可用则通过 Origin 完成，并报告第一个打断链条的 PR。已验证的 PR 如果位于未验证的 PR 之上，就会等待，因为合并它会把缺口拉到它下面。

下一步：[睡觉时也让工作继续跑](./07-overnight.md)。
