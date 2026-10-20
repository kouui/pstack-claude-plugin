# 让它成为你的

poteto-mode 是某一个人的风格。底下的 playbook、路由和模型角色同样适用于你自己的风格。本页介绍如何生成个人模式、从一次会话中记录经验、修复仓库让 agent 不再重复犯错、编写聚焦的 skill，以及在信任 skill 改动之前先测试它。

起步要比你想的小。第一天不需要很多 skill，甚至不需要整个插件。直接平实地提问，观察 agent 在哪里失败；同一个失败出现第二次时，再添加一个 skill 或一项检查。

## 用 `/automate-me` 生成你自己的模式

```text
/automate-me
```

你不需要描述自己的风格，因为 [`/automate-me`](../../../skills/automate-me/SKILL.md) 会从你的历史记录中读取它。它会挖掘当前工作区里你最近的 transcript，找出反复出现的偏好，包括你希望的回复方式、委派、验证、代码、文字和流程，然后询问你哪些模式真正代表你。它按 Claude Code 的 skill 格式起草 `.claude/skills/<your-name>-mode/SKILL.md`，把草稿交给 [`/unslop`](../../../skills/unslop/SKILL.md) 处理，并从 worktree 开一个 PR，让你像审查其他改动一样审查它。

每当你的习惯有变化时，就再运行一次：

```text
/automate-me update my mode skill with everything since its last edit
```

更新模式只会挖掘自 skill 上次修改以来的历史。它会保留你没有推翻的规则，修订有新证据支持的规则，并且只为新出现的模式添加章节。

## 用 `/reflect` 记录会话的经验

在某个让你有所收获的任务结束后，立即运行：

```text
/reflect that took way too long. capture what we learned so the next run doesn't repeat it.
```

[`/reflect`](../../../skills/reflect/SKILL.md) 会把 transcript 发给三个并行的审查者，然后由一个汇总者把提案分成 `Accepted`、`Rejected` 和 `Backlog` 三类，并在改动任何 skill 之前等待你的批准。只有当某条提案会改变未来的决策时才批准它。一次奇怪的会话只是个例子，不是规则。

## 用 `/correct` 修复环境

当你一再因为同一个错误纠正 agent 时，修复应该放在仓库里，而不是你的下一条提示里。按照修复的牢固程度给方案排序：

1. 用架构或更好的数据结构，让错误不可能发生。
2. 用类型，或用一个错误信息会直接指出修复方法的 lint 或 CI 检查来阻止它。
3. 用测试捕获它。
4. 写成文档或 agent 规则。没有任何东西会因为 agent 跳过规则而失败，所以这一项排在最后。

人工审查不在这个列表里。如果审查者必须在每个 PR 上都抓住同一个错误，那正是这一修复要消除的问题。[`/correct`](../../../skills/correct/SKILL.md) 负责完成这项工作：

```text
/correct agents keep calling the database client directly instead of going through the repository layer
```

它会读取最近的提交、revert、review 评论，以及解释变通做法的注释，然后把错误归成若干类。一个类别在出现两次之后才算数。它会以能够奏效的最高层级修复最频繁的几类，每类一个提交，并证明每个新检查都能在真实的过去错误上失败。它还会在 agent 指令文件中维护一张表，把每条规则与执行它的机制配对，这样没有任何机制执行的规则就会以重复出现的形式暴露。回复会列出每个类别、其证据、所选的层级，以及为什么更高的层级行不通。

不带参数运行时，它会自行从历史中找出这些类别。`/reflect` 和 `/correct` 各自分工：`/reflect` 从一次会话中改进 skill；`/correct` 改变仓库，使某类错误无法再次出现。当修复需要一个新边界时，可搭配 `/architect` 使用。[在多个项目中并行运行](./07-overnight.md#run-many-projects-in-parallel) 中有一个可以在整个仓库范围内同时完成两件事的提示。

## 编写聚焦的 skill

当你已经知道想要固化的工作流时：

```text
/poteto-mode write a skill for verifying database migrations in this repo
```

编写 skill 符合 [编写或修改 skill 的 playbook](../../../skills/poteto-mode/playbooks/authoring-a-skill.md)。该 playbook 会按 Claude Code 的 skill 格式编写 SKILL.md，验证 frontmatter 和链接，并通过 Opening a PR playbook 交付结果。面向 agent 的文字要求比面向人的文字更高，因为一句没用的话会变成某个未来 agent 会遵循的指令。让 playbook 守住这个标准，不要徒手编写 `SKILL.md`。

有两类特殊情况有各自的生成器。如果一个 skill 必须驱动你的应用并证明其行为，它就是验证型 skill，应改用 [`/create-verification-skill`](../../../skills/create-verification-skill/SKILL.md) 和 [`/maintain-verification-skill`](../../../skills/maintain-verification-skill/SKILL.md)。如果一个 skill 用模型检查器、证明或其他验证器检查代码的某项属性，它就是形式化验证 skill，应改用 [`/create-formal-verification-skill`](../../../skills/create-formal-verification-skill/SKILL.md) 和 [`/maintain-formal-verification-skill`](../../../skills/maintain-formal-verification-skill/SKILL.md)。[验证并交付](./06-verify-and-ship.md#prove-a-property-with-a-formal-verification-skill) 涵盖这两类。

## 用 `/technical-writing` 按标准写文档

skill 不是你交付的唯一文字。对于文档、RFC、readme、PR 描述和提交信息：

```text
/technical-writing review the readme changes
```

[`/technical-writing`](../../../skills/technical-writing/SKILL.md) 应用一套分层标准。它的目标是让疲惫的工程师第一次读就能理解的文字。它先选定文档的模式（教程、操作指南、参考或解释），然后逐句处理：谁做什么、每句只表达一个想法、没有任何句子能有两种读法。可以用它来审查你或 agent 刚写的内容，也可以在请求文档时直接点名使用。

## 盲测 skill 改动

skill 的编辑会影响之后的每一次会话，因此要把它当作一次实验来测试。采用别人的 skill 也是同样道理。在保留它之前，先确认它确实能让 agent 在你的工作上表现更好。

```text
/poteto-mode run the eval playbook on this skill change. same task for both variants, candidates stay blind.
```

当某个 skill 总是出错，而你知道它应该做什么时，可以在一个任务中同时修改并测试它：

```text
/poteto-mode update the review skill so it flags missing migrations, and eval the change.
```

事先要求 eval，可以让修改保持诚实。只根据一次糟糕会话写出的修复往往会对那一次会话过拟合，经过多次修改之后，skill 会逐渐漂移。eval 可以在发布之前发现这种漂移。

[Eval playbook](../../../skills/poteto-mode/playbooks/eval.md) 围绕一种失败模式构建，即观察者效应。知道自己正在被评估的 agent 会表现得不一样。因此，候选 agent 会在经过清理的目录中接到一个看起来很自然的任务，永远看不到 "eval" 或 "candidate" 这些词，也永远不知道彼此的存在。由一个评审者用中立标签为所有输出打分；对于链式跟随的评分，依据的是每个候选者实际读取了哪些文件，而不是它们自己声称的内容。

在接受结论之前，请亲自阅读每一份输出。如果你不同意评审者的判断，先怀疑评分标准，再怀疑你自己的判断。

## 用 `/make-bot-ui` 构建 bot UI

有一项情境性的 skill 面向 Grok Bot 用户。[`/make-bot-ui`](../../../skills/make-bot-ui/SKILL.md) 会构建一个小页面，其按钮通过 webhook 例程唤醒 bot。例如，你可以滑动浏览一个审查队列，每次滑动都让 bot 对该条目采取行动。你机器上的一个服务器持有 webhook 的发送方密钥，因此密钥永远不会到达浏览器或聊天中。该 skill 还涵盖通过 Tailscale 暴露该页面。

**陷阱:** 不要因为某个 skill 出了问题就在任务中途修改它。请在它自己的 PR 中修复，让任务继续推进。与功能工作混在一起发布的 skill 改动对 review 不可见，也无法被评估。

下一步: [配方与陷阱](./10-recipes-and-pitfalls.md)。
