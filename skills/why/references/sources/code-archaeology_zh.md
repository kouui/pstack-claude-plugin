# 代码考古（git + 仓库内容）

## 此来源包含的内容

- 提交历史（提交信息、日期、作者、diff）
- PR 描述、review 评论和讨论串（通过 `gh`）
- 代码内注释、TODO、FIXME、弃用说明
- ADR（架构决策记录），如果仓库有保存的话
- 测试，其名称和断言通常记录了促成某次修改的边界情况
- 同一提交中一起修改的相关文件（共同变更信号）
- 仓库中的 CHANGELOG 条目和发布说明
- 提交信息和 PR 正文中的 issue/ticket ID

这是最可信的来源，与代码直接相关，也是最完整的。所有经过仓库的内容都应该在这里。

## 如何搜索

展开种子提交列表：

```bash
# 追踪重命名的历史
git log --follow --oneline -- <file>

# Pickaxe：新增或删除了这段确切文本的提交
git log -S '<exact_string_from_code>' -- <file>

# 或者按模式搜索
git log -G '<regex>' -- <file>

# 每一行是谁写的、何时写的
git blame -L <start>,<end> <file>

# 单个提交的完整 diff
git show <hash>

# 两个节点之间影响该文件的提交
git log <old>..<new> -p -- <file>
```

查找仓库外的文档：

```bash
# ADR 通常位于 docs/adr/ 或类似目录
rg -l -i 'architecture.decision' --glob '*.md'

# 目标文件附近的 TODO 和 FIXME
rg -n -C2 '(TODO|FIXME|HACK|XXX|NOTE)' <target_file>

# 相关测试
rg -l '<symbol>' --glob '*test*'
```
