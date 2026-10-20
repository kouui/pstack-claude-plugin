### 发布

**你对落地的内容负责。逐个独立验证每个 PR，只从根部落地已验证的运行段，然后不再插手这个 stack。**

1. **先确定 forge，再在落地前独立验证每个 PR。** GitHub CLI（`gh`）是默认选项。如果 `command -v origin` 成功，并且 Origin 能解析该仓库，则所有 PR 操作都使用 `origin pr`。否则继续使用 `gh`，并记录这一回退。绝不要求使用 Graphite（`gt`）。每个 PR 配一名验证者，每名验证者都在自己的本地 git worktree 和分支中工作，它配合项目自己的验证 skill（`verify-<app>`，见 `/create-verification-skill`）或项目自己的脚本，以用户的方式运行该变更，比较 parent 与 head，并在该 PR 上发布 `PASS`、`PASS+NOTES` 或 `FAIL`。只有未编写该代码的 agent 给出的结论才算数，因为作者检查的只是他想构建的东西。绿色的 CI 和通过的 bot 审查都不是结论，因为两者都没有以用户的方式运行该变更。
2. **只落地从最底部开始的连续已验证运行段。** 从最低的未合并 PR 向上走，在遇到第一个没有 `PASS` 或 `PASS+NOTES` 的 PR 时停止。它下面的 PR 构成运行段，最上面那个是上限。已验证的 PR 如果位于未验证的 PR 之上，就要等待，因为它没有下面的代码就无法落地。如果某个 PR 的分支既不包含也不需要下面的代码，它可以移到 trunk 上并单独落地。但它需要先获得新的结论，因为它的旧结论是在下面有那些代码的情况下得出的，而 trunk 上没有这些代码。
3. **重新检查结论是否仍然描述当前代码。** 结论针对的是一个 patch，而 chain rebase 会把其上的每个 PR 移到新的 SHA，且不会让任何检查失败。在信任旧结论之前，比较结论 SHA 上的 `git patch-id` 与当前 head 的 patch-id。当两者的 patch 只在测试、文档或 lint 配置上不同时，只要某条 lane 所运行的内容在两个 SHA 上构建结果相同（排除噪声），该 lane 的结果依然成立。噪声是指：在结论 SHA 的两次构建之间也会出现的差异，或嵌入的 commit SHA。对每一类噪声，都要列出其涉及的文件，以便读者核对这一判断。变更本身仍然需要新的检查和审查，因为旧结论从未见过这一变更。没有构建输出的 lane（例如开发服务器）无法证明这一点，因此需要重新运行，并重新验证其他已发生漂移的内容。
4. **自底向上，每次只落地一个 PR。** 始终合并运行段中最低的未合并 PR，使用 `origin pr merge <pr>` 或 `gh pr merge <pr> --squash`。在 Origin 上，普通命令还会合并你所指定 PR 之下的所有 PR；在 GitHub 上，base 仍为其 parent 分支的 PR 会合并到该分支，而不是 trunk。通过读回 PR 来确认结果（`origin pr view <pr>` 或 `gh pr view <pr> --json state,mergedAt,baseRefName`），因为当合并队列只是把 PR 排入队列时，合并命令也可能报告成功；已请求的 auto-merge 也不等于已合并。已落地的 PR 应显示为已合并，且 base 为 trunk，而不是其 parent 分支。只对结论覆盖其当前 head 的 PR 启用 `--auto`（Origin 的 merge-when-ready 或 GitHub 的 auto-merge），因为它会落地任何已就绪的 head。合并落地后，将 base 为已合并分支的 PR 改指向 trunk（`origin pr edit <pr> --base <trunk>` 或 `gh pr edit <pr> --base <trunk>`），把它的 patch 移到合并结果之上，并使用 `--force-with-lease` 推送。结论覆盖的是 patch，而不是 commit，因此步骤 3 的 patch-id 规则决定该结论是否仍然有效。在其检查通过且结论仍然有效后再合并它。
5. **落地期间不要动这个 stack。** 步骤 4 中对下一个 PR 的移动，是对某个分支或 base 的唯一改动，因为合并时会落地该时刻分支和 base 上的任何内容，无论是否有结论。在 GitHub 上，在 `/loop` 下使用 `playbooks/babysit.md` 中的排队监视器（`scripts/watch-pr/watch-pr --queued-stack --stack-prs <bottom-to-top>`）观察运行，直到它在上限处报告 `COMPLETE`。在 Origin 上，每次 `origin pr checks <pr> --watch` 返回时，都用 `origin pr view <pr> --checks` 重新读取当前 PR，直到上限显示为已合并。当父 PR 合并后，base 移到 trunk 是预期情况，不是需要修复的问题。如果合并停滞，先查明原因，再做任何改动。
6. **在上限处停止。** 扩展运行段需要新的结论，因此这属于新一轮的步骤 1。

**回复：** 已验证的运行段及其上限，每个 PR 的结论及其产出者，已落地的内容以及你如何确认，以及下一个缺口需要什么。
