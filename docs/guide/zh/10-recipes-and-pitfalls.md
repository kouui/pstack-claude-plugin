# 配方与常见陷阱

值得直接复制的提示词，然后是每个人都会犯一次的错误。请替换成你自己的路径和完成条件。这些配方刻意保持随意。实际输入时大家就是这样写的，而 skill 也能正确理解意图。

![她在品尝一道做好的菜，机器人照着配方盒做菜，柜台上方钉着写有 /how、/tdd 和 /loop 的卡片。](../images/recipes.jpg)

## 了解一个陌生的子系统

```text
use /how first to understand how this initialization works. then use /why to figure out why it broke recently.
```

先看机制，再看历史。每个 skill 的报告都会告诉你它搜索了哪些来源，因此你能知道答案的依据是什么。

## 动代码之前先复述一份嘈杂的报告

```text
/poteto-mode read this thread. restate the underlying issue in your own words, in plain english. don't change any code yet.
```

误读会在复述中暴露出来，而纠正它只需要一条消息。在 agent 说出它自己的理解之前，先把你自己的猜测藏起来。`/in-your-own-words` 只要求复述，不运行任何 playbook。

## 在任务开始前校准目标

```text
/sharpen write the onboarding doc for the new billing service.
```

agent 会询问专家会如何完成这项任务、你真正想要的是什么，以及你的请求是说得太多还是太少。它会保留一项外部检查，确认结果是正确的，然后再开始工作。

## 先试做，再做选择

```text
/poteto-mode prototype a few options for the settings layout. put them behind a switcher and send me screenshots of each.
```

你从能运行的东西中做选择，而不是从文字描述中选。agent 会在过程中自行回答布局和耗时方面的问题。

## 把定下来的设计变成计划

```text
/poteto-mode turn this design into a plan. small verifiable PRs, each with its own proof.
```

设计定下来之后再提出。计划才是交付物，而且它会指明将执行它的 playbook。

## 就设计征求第二意见

```text
ask /arena for a second opinion on this thread and our approach
```

你当前的设计会成为众多候选方案中的一个，而综合结果会告诉你，评审团是找到了更好的方案，还是确认了你原有的方案。与一个难以逆转的设计的代价相比，这点成本很小。

## 并行检查相互独立的分片

```text
/swarm check every package under packages/ against its check.sh. one worker per package. one report.
```

每个 worker 负责一个包。父级会等待所有分片完成，并返回一份 `PASS`、`ISSUES` 或 `BLOCKED` 报告，而不是原始的 worker 输出堆。

## 以怀疑的态度审查一个分支

```text
/interrogate the whole branch, but skeptically. don't change anything yet. no nitpicks unless it's an actual bug or regression in behavior.
```

这些限定词起了实际作用。"don't change anything yet"（先不要改任何东西）让它保持只读，而"不挑刺"的规则会预先过滤噪音，使 `Act on` 类的发现值得你花时间。

## 通过失败的测试修复 bug

```text
/poteto-mode repro the duplicate write first. if there's a cheap test path, /tdd it. then fix and rerun.
```

"if there's a cheap test path"（如果有低成本的测试路径）这个条件很重要。强行通过脆弱的 mock 来跑测试，证明力很弱，而运行真实命令更可靠。playbook 允许它直接这样说。

## 复现并用证据修复报告

```text
/poteto-mode repro this with /verify-<app>. if it repros on main, fix it and show me a video as proof.
```

"if it repros on main"（如果在 main 上能复现）让运行可以在 bug 已经消失时提前停止。视频让你在阅读 diff 之前就能检查修复效果。

## 证明测试很少能覆盖到的性质

```text
/create-formal-verification-skill the retry queue. a message must never be delivered twice.
```

在声明的范围内，你会得到一个在所有情况下都被检查过的性质；如果代码本身已经破坏了这个性质，你会得到一个用你代码的术语描述的反例；另外还会得到一个项目 skill，下一个 agent 在修改该代码时会运行它。

## 在发布前核实一个数字

```text
/benchmark-checklist vet this 40% speedup before it goes in the pr description
```

你会得到"更快"、"更慢"、"没有可测量的差异"或"结论不确定"之一，并附带运行次数、波动范围，以及限制这个数字的因素。

## 停止重复犯同样的错误

```text
/correct agents keep adding new config flags without registering them in the schema
```

修复会以架构、类型、lint 或测试的形式落到仓库里，因此下一个 agent 就无法再犯同样的错误。

## 只问怎么做，不开始干活

```text
/poteto-help how do i get poteto-mode to stay on every turn?
```

你会得到一个回答、一条可以发送的提示词，以及指向源码的链接。在你发送那条提示词之前，什么都不会运行。

## 离开时让运行保持诚实

```text
im going to bed, keep going autonomously until every fixture passes. do not stop. keep a decision log i can audit in the morning.
```

完整的约定见[长时间运行页面](./07-overnight.md)。如果任务和完成条件已经在对话中，简短的写法也能生效。

## 纠正偏离方向的运行

纠偏提示词只需一行：

```text
i said the goal is to repro. i did not ask for a fix yet.
```

```text
apply prove it works. show me the real output, not the build log.
```

```text
/unslop that, no emdashes
```

你很少需要多说几个字，只需要说出正确的名字。[原则页面](./08-principles.md)就是这套词汇表。

## 用大白话获得回复

```text
/bro
```

这就是整个提示词。[`/bro`](../../../skills/bro/SKILL.md) 会像一个人对另一个人说话那样复述上一条消息，不用行话，而且更简短。当一条回复技术上很完整，而你仍然不知道它说了什么时，就用它。

## 补看你离开时仍在运行的对话

```text
/ketchup
```

[`/ketchup`](../../../skills/ketchup/SKILL.md) 会用大白话告诉你，自上一条消息以来发生了什么变化，然后列出每一个正在等待你决定的调用，附上它的选择、依据和风险。报告期间它不会改动任何东西。

## 常见陷阱

- **在提示词里列举 skill。** "use /how then /architect then /arena" 会打乱 playbook 已经排好的步骤。请说明目标和约束。只有在需要覆盖默认行为时，才点名某个 skill。
- **模糊的完成条件。** "make it better" 没有给 `/loop` 留下可以检查的东西。请给出一个能通过或失败的命令或产物。
- **一开始就说出你对原因的判断。** agent 会在你指向的地方搜索。请先让它复述问题，然后再分享你的猜测。
- **直接采用第一个设计。** 一次尝试就会锁定模型最先想到的形状。请要求原型或 `/architect`，并基于证据做选择。
- **打磨一份抽象的计划。** 对一份背后没有代码的计划进行对抗式评审，会凭空编造永远不会发生的风险。先用原型解决悬而未决的问题，再评审已经构建出来的东西。
- **在同一个 worktree 里并行运行 agent。** 它们会互相覆盖，没人能分清哪个 agent 改了什么。请给每个 agent 一个独立的 git worktree 和分支，或者说明"每次尝试使用独立的 worktree"。
- **在信任循环之前就让它循环。** 一个无法验证自身工作的循环，只会让未经检查的工作变得更快。先让验证 skill 跑起来。
- **相信未经核实的数字。** 预热的缓存或被跳过的代码路径，都可能伪造出加速效果。在数字用于任何地方之前，先运行 `/benchmark-checklist`。
- **手动纠正同一个错误。** 在聊天中的纠正只对一次运行有帮助。`/correct` 会修复仓库，让之后的运行不会重复同样的错误。
- **用 `/arena` 来做覆盖。** `/arena` 会重复同一份设计或代码需求，然后选定基础版本并嫁接最好的部分。`/swarm` 负责对分片或预先声明的赛跑分支进行分区，并汇总成一份报告。
- **照单全收每一条评审意见。** 机器人和人类都会在同一列表里提交真正的发现和噪音。`/interrogate` 会把发现分为"采取行动"和"驳回"两类，并附上理由，你可以朝任一方向推翻它。
- **把 `auto` 当作模型 slug。** `auto` 和 `inherit-parent` 的意思是"省略 model 字段，让子 agent 继承父级对话的模型"。[设置](./01-setup.md)页面介绍了各个角色。
- **看到构建通过就认定成功。** 构建只能证明代码能编译。请要求真实的命令、流程、存储的值或性能分析结果，并期望在回复中看到证据。
- **手写 `SKILL.md`。** 请通过[编写或修改 skill 的 playbook](../../../skills/poteto-mode/playbooks/authoring-a-skill.md) 来完成，这样才会进行校验和评审。

以上就是本指南的全部内容。如果你是跳过前面直接看到这里的，请回到[设置](./01-setup.md)，完整地跑一个真实任务。这些习惯是通过使用养成的，而不是通过阅读。

返回[指南索引](./README.md)。
