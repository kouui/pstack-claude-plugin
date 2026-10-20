### 提交 PR

在每个其他 playbook 结束时调用。

**Worktree。** 从 main 分支的 git worktree 中工作。子代理会继承该 worktree。同一分支上的多个 `Task` 调用各自使用一个 worktree，或在两次调用之间执行 `git fetch && git reset --hard origin/<branch>`。分支不干净且含有无关工作：将改动 patch 出来，新建 worktree，再应用 patch。worktree 混乱：从 main 重置，然后以最小改动重做。

**Commits。** 大胆提交。在提交 PR 之前，将改动 rebase 成小而有序的 commit。每个 commit 都是一个未来的 PR：可以独立合并，并按顺序讲述故事。如果修复属于刚刚做出的 commit，则 amend。如果修复可以单独拆出，则新建 commit。commit 正文不要重复主题行的内容。

**PRs。** 在提交之前，对 diff 运行 `/deslop`，在进入评审之前运行 `/no-comments`。所有 PR 标题、描述和 commit 正文都使用 `/technical-writing` 编写（覆盖除 Diátaxis 之外的每一层，并应用 STE 词汇规则：每个动作一个词，保留冠词，能用普通动词时不用 `-ing` 形式），然后应用 `/unslop`。

**Titles。** 使用 Conventional Commits 格式 `type(scope): subject`，结尾不加句号。type 例如 `feat`、`fix`、`docs`、`refactor`、`test`、`chore` 或 `perf`。scope 是所属领域，例如 `pstack` 或 `poteto-mode`。subject 要简短、使用祈使语气，并且当某个真实符号承载了这次改动时，要写出该符号的名称，例如 `fix(pstack): retarget opening-a-pr babysit trigger`。

**Descriptions。** PR 正文是给已经看过 diff 的评审者用的一分钟简报，不是实验笔记。它说明改动为什么存在、不包含什么、可能会破坏什么，以及你如何证明它有效。句子要短且简单，标识符要少，不要写成大段文字。正文会成为 squash commit 的正文，因此该 commit 应控制在约 40 行以内。使用 `##` 标题，不要使用加粗的引导文字，并按以下顺序排列；如果某一节没有内容可写，就省略该节：

- `## Why`：问题和方案，1–3 句短句。不写 SHA、rebase 的来源关系，也不写“基于 main”之类的前言。
- `## What changed`：1–3 条简短的要点。只有当某个真实符号或路径承载了这次改动时，才写出它的名称。重命名或重定向时，两侧都要写出。
- `## Scope`：必须包含。1–3 条简短条目，说明涵盖什么，以及有意不涵盖什么（后续工作、已知缺口）。不写符号、路径，也不逐文件列举。
- `## Tradeoffs`：只写评审者否则会追问的、被否决的备选方案。如果没有真正的选择，则省略。
- `## Blast Radius`：1–2 句，说明它影响了谁或什么，以及为什么这样是安全或有风险的。如果 main 处于红色状态，说明让它保持红色的代价。
- `## Verification`：1–3 条要点，每条都是一个真实的运行路径及其结果。性能相关：给出一个主要数字及其单位，格式为 `before → after`，并链接到 arena 或 swarm 目录以查看其余数据。不要写样本量方法论、swarm 复述或指标表格。

各节之后，当视频或截图能证明某个论点时，附上它们。完整 SHA、swarm 或 arena 分支的复述、修正杠杆的长篇说明、逐文件检查清单，以及 “CLEAN” 结论，都应放在链接的产物中，而不是正文里。

**Forge。** 在第一次执行 PR 操作之前确定代码托管平台，并在 create、edit、view、watch 和 merge 中保持该选择。GitHub CLI（`gh`）是默认选项。如果 `command -v origin` 执行成功，并且 Origin 能解析该仓库，则优先使用 `origin pr ...`。如果 Origin 不存在或无法解析该仓库，则继续使用 `gh`，并记录回退情况。不要求使用 Graphite（`gt`）。

**Built-in PR tool。** 如果本次运行提供了内置 PR 工具，则通过它来创建、编辑、重定向和标记为就绪，绝不通过代码托管平台的 CLI。通过 CLI 创建的 PR 会遗漏该工具所跟踪的内容。对于该工具不覆盖的所有操作，使用已确定的代码托管平台；如果本次运行没有此类工具，则所有 PR 操作都使用已确定的代码托管平台。

**Size and stacks。** 优先选择五个窄的 PR，而不是一个大的 PR。stack 是一条以基础分支串联起来的链。根 PR 指向 trunk。每个子分支都 rebase 到其父分支的精确最新提交上，且其 PR 指向父分支。当父分支被合并后，按照 `playbooks/shipping.md` 第 4 步，将其子分支移动到 trunk 上。如果没有内置 PR 工具，则根据已确定的代码托管平台，使用 `origin pr create --status open --base <parent-branch>` 或 `gh pr create --base <parent-branch>` 创建子 PR，并使用 `origin pr edit <pr> --base <parent-branch>` 或 `gh pr edit <pr> --base <parent-branch>` 重定向。只有独立的工作才从 trunk 分支出来。在进行大量 stack 工作之前，先 rebase 到 trunk 上。

**Readiness。** 所有 PR 都直接以就绪状态打开，绝不以草稿形式打开。内置 PR 工具可能默认创建草稿，因此每次通过它创建时都要设置 `draft: false`。使用 Origin 时，传入 `--status open`。使用 `gh` 时，省略 `--draft`。如果某个 PR 仍以草稿形式打开，则通过 PR 工具将其标记为就绪，或根据已确定的代码托管平台运行 `origin pr ready <number>` 或 `gh pr ready <number>`。在提及 PR 状态之前，先运行 `origin pr view <number>` 或 `gh pr view <number>`。

**Babysit。** 打开 PR 并不会启动 babysit。发布 URL 后继续构建。先完成该阶段或整个 stack。只有在用户于整个 stack 建立之后要求时，才运行单独的 babysit 轮次。为每个新 PR 启动 babysit 会拖慢构建，并把检查花在之后的波次会重新开始的提交上。当反馈偏离原意时，要予以反驳。

打开 PR 的子代理需要运行 `interrogate`、`/deslop` 和 `/no-comments`，并发布 URL。除非它是 Autopilot-full 或 Autopilot-stack 的负责人，否则它会返回给父代理，而不进行 babysit。该负责人的任务简报会指定 babysit 循环，这也是 `playbooks/babysit.md` 所等待的请求。负责人在发出代码就绪报告之后启动该循环，并根据其 playbook 报告 merge-ready 或 STACK-READY；在整个 stack 构建完成之前，本文件及 `playbooks/babysit.md` 中限制 babysit 的规则对其不适用。
