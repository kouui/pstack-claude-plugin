### 清理 worktree 和模拟器

1. 快照与审计。记录 `df -h /`，然后运行 `scripts/worktree-audit.sh [-b <pr-base-ref>] [repo-path]`（principle-build-the-lever）。不带 `-b` 时，它会对照 `origin/HEAD` 检查 `MERGED`，因此当 PR 指向其他分支时，请传入 `-b origin/<branch>`。它从 `git worktree list` 读取路径，从不手动输入，因为手动输入的 `myrepo-worktrees/x` 会漏掉位于 `.cursor/worktrees/myrepo/x` 的 worktree（principle-encode-lessons-in-structure）。它会按大小、年龄、合并状态、未提交工作、PR 状态以及最近触及它的聊天对每个 worktree 进行分类，然后建议一个分组。转录扫描较慢，因此应放到后台运行。
2. 分组只是建议，不是许可。固定的和活跃的聊天才是真正的依据（principle-prove-it-works）。从用户或侧边栏获取这一集合，并逐一核对每个候选项。工具曾把用户固定的 worktree 标记为 `safe`，因此以固定集合为准。
3. 删除前核实使用情况。对于每个 `verify-recent-chat` 行，或任何你存疑的项，派生子代理去阅读转录，报告该聊天是否已固定或仍在进行，以及它涉及哪些 worktree（principle-guard-the-context-window，转录内容量大）。固定的聊天会通过后台子代理在同级 worktree 中生成 arena 和 repro 树，即使这些名称从未出现在侧边栏中，它们仍在使用中。请按照 Subagents 部分的说明，为每个子代理选择合适的推理级别以及快速或标准速度。
4. 对不可逆的损失暂停。`wip:N` 表示 N 个已跟踪的未提交编辑。先展示差异并获得决定，因为移除干净的 worktree 可以从其分支恢复，但未提交的工作将永久丢失。`scratch:N` 是未跟踪的临时文件，可以安全删除，但需列出文件名。按照 Autonomy 的规定，干净、已合并且未在使用的项可以继续。`wip` 和使用中的项必须暂停。
5. 清理已确认的集合。对每个路径，执行 `git worktree remove --force <path>`。如果目录因忽略的构建产物而残留，则执行 `rm -rf` 删除，然后执行 `git worktree prune`。分支引用会保留，因此不会丢失任何提交。使用 `df -h /` 确认，并重新列出。
6. 模拟器及其他可回收项。模拟器通常是下一个最大的收益来源。`xcrun simctl --set testing delete all`（XCTestDevices 克隆）、`xcrun simctl delete unavailable`，以及 `xcrun simctl runtime list` 后执行 `runtime delete <id>` 删除旧运行时。需要时还可以清理：Xcode `DerivedData` 和 `iOS DeviceSupport`、`~/Library/Application Support/Cursor`（`state.vscdb.backup`，以及 `snapshots/roots/<root>` 中，以你打开为工作区的文件夹命名的 `<root>` 会膨胀），以及包缓存（pnpm、uv、brew、yarn）。只清理用户未说明需要保留的缓存。

这是唯一一个在删除用户状态时没有代码审查来捕捉失误的 playbook，因此上述各道关卡就是审查。

**回复：** 提供清理前后的 `df -h /`、已清理的 worktree，以及每个被保留项的一行原因（被哪个聊天使用，或存在未提交工作）。
