### Autopilot-stack

**你负责整条 stack，但永远不负责落地合并。向操作者交付一条线性的、按 base 串接的 PR 栈，供其评审并落地。** **Autopilot-full** 继续生效，除非涉及已验证的工作如何离开本地。每个 tick 都从 trunk 重新读取两份 playbook，两者有差异时以本 playbook 为准。

1. **运行 Autopilot-full 的 owner 机制，但去掉落地环节。** Owner 以 STACK-READY 代替 merge-ready 报告。他们绝不 rebase 到另一分支上（包括 trunk），也绝不 retarget、merge、关闭 PR 或开启 auto-merge，因为根节点掌管链的拓扑，操作者掌管落地。当某个条目依赖另一个条目时，其 owner 从该条目的分支创建分支，而不是从 trunk 创建。
2. **让拓扑只有一个写入者。** Owner 只推送自己的分支，并报告每个分支的 tip 及其预期父分支。只有你可以 rebase 或 retarget 链中的某个环节，因为一条链上有两个写入者会相互抵消对方的修改。将每个干净的 PR 按依赖顺序追加，并把其 base 设为父分支（`origin pr edit <pr> --base <parent-branch>` 或 `gh pr edit <pr> --base <parent-branch>`）。绝不要求使用 `gt`。Owner 将你的推送视为其新的 base 和 lease，若发生冲突，则由其解决自己负责的部分，再交由你推送。
3. **在根节点吸收漂移。** 在落地开始之前，对每个环节应用 Autopilot-full 第 5 步的测试，并在其要求时从下往上 rebase 整条链。然后逐个环节应用 patch-id 规则，因为每个环节的 base 就是其父节点。
4. **交付整条链，然后不再插手。** 每个环节在其 PR 中附带其结论。落地开始后，除 `playbooks/shipping.md` 所允许的落地后 retarget 外，任何人都不得写入这条链，因为第二个写入者会破坏进行中的落地。落地过程中出现的问题应经由你上报给操作者，任何修复都须等待操作者的批准。

**在两种 autopilot 之间选择。** 当 PR 相互独立且已授予落地权限时，选择 Autopilot-full。当操作者希望在落地前进行评审、工作有先后顺序或相互耦合，或未授予合并权限时，选择 Autopilot-stack。

**回复：** 依据 **ketchup** skill 向操作者汇报情况，包括指向 stack 根节点和 tip 的链接、每个环节一行的结论，以及任何被搁置或排除的项目及其原因。
