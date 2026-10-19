# 搜索笔记

搜索让用户按标题或正文文本查找笔记、查看匹配的笔记，并区分"没有匹配结果"与"搜索不可用"。

## 子功能

- `search-open` 从每个受支持的浏览器入口打开搜索。
- `search-match` 返回标题和正文的匹配结果，且不改变笔记数据。
- `search-open-result` 在笔记编辑器中打开一个结果。
- `search-empty` 对没有匹配结果的查询显示完整的空状态。
- `search-clear` 清除查询并恢复最近笔记视图。
- `search-cli` 从终端返回相同的匹配笔记。

## 如何进入（用户视角）

- 选择浏览器工具栏中的 `Search` 按钮。
- 当焦点不在可编辑字段内时，在浏览器中按 `/`。
- 在终端中运行 `notes search <query>`。

## 使用 control-notes 操作

前置条件：

- Notes 在 `http://127.0.0.1:4173` 上运行正常。
- 一次性数据目录中包含标题为 `Quarterly plan`、正文为 `Draft budget` 的笔记。
- `control-notes doctor` 报告的 URL 和数据目录符合预期。

- **工具栏入口。** 选择 `Search` 按钮。运行 `control-notes browser click --role button --name "Search"`。名为 `Search notes` 的对话框出现，焦点位于其搜索框中。
- **键盘入口。** 关闭对话框，让页面获得焦点，然后按 `/`。运行 `control-notes browser press --key "/"`。同一对话框出现，且页面不会输入斜杠。
- **标题匹配。** 输入 `quarterly`。运行 `control-notes browser fill --role searchbox --name "Search notes" --value "quarterly"`。`Search results` 列表包含 `Quarterly plan`，且不包含 `Grocery list`。
- **正文匹配。** 将查询替换为 `budget`。运行 `control-notes browser fill --role searchbox --name "Search notes" --value "budget"`。结果 `Quarterly plan` 仍然可见，并带有正文匹配摘录。
- **打开结果。** 选择 `Quarterly plan`。运行 `control-notes browser click --role link --name "Quarterly plan"`。对话框关闭，编辑器标题显示 `Quarterly plan`。
- **空状态。** 重新打开搜索并输入 `volcano`。运行 `control-notes browser fill --role searchbox --name "Search notes" --value "volcano"`。搜索完成后，名为 `No matching notes` 的状态出现。
- **清除查询。** 选择 `Clear search`。运行 `control-notes browser click --role button --name "Clear search"`。搜索框为空，`Recent notes` 区域替换结果列表。
- **CLI 匹配。** 从终端搜索。运行 `control-notes cli -- notes search "quarterly" --format json`。退出码为 `0`，标准输出包含一个对象，其标题为 `Quarterly plan`。
- **CLI 未匹配。** 搜索一个不存在的值。运行 `control-notes cli -- notes search "volcano" --format json`。退出码为 `0`，标准输出为 `[]`。
- **证明材料。** 捕获有结果时的状态。运行 `control-notes browser snapshot --aria --path artifacts/search/results.aria.txt` 和 `control-notes browser screenshot --path artifacts/search/results.png`。两个产物都要能标识 Notes、查询词以及 `Quarterly plan`。

## 注意事项

- 当编辑器或搜索框有焦点时按 `/`，会输入文本，而不是打开搜索。
- 结果在短暂防抖后更新。应等待结果列表或空状态出现，而不是使用固定的 sleep。
- 除非用户启用 `Include archived`，否则已归档的笔记会被排除。
- CLI 默认输出为人类可读格式。进行稳定断言时请使用 `--format json`。
- 打开结果会改变浏览器状态。在验证另一个查询之前，请重新打开搜索。
