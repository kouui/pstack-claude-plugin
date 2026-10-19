# 创建笔记

Create note 允许用户从浏览器或 CLI 保存一条带标题的笔记，取消未完成的草稿，并在面向用户的第二个视图中确认已保存的笔记。

## 子功能

- `create-open` 从每个浏览器入口打开空白编辑器。
- `create-save` 保存标题和正文。
- `create-cancel` 丢弃未完成的浏览器草稿。
- `create-cli` 从终端创建相同结构的笔记。

## 如何进入（用户视角）

- 选择浏览器工具栏中的 `New note` 按钮。
- 当焦点不在可编辑字段内时，在浏览器中按 `n`。
- 在终端中运行 `notes create --title <title> --body <body>`。

## 使用 control-notes 操作

前置条件：

- Notes 在 `http://127.0.0.1:4173` 上运行正常。
- 没有标题为 `Release checklist` 的笔记。
- `control-notes doctor` 报告的 URL 和一次性数据目录符合预期。

- **打开编辑器。** 选择 `New note`。运行 `control-notes browser click --role button --name "New note"`。一个名为 `Note editor` 的表单出现，焦点位于 `Title` 文本框中。
- **输入内容。** 输入标题和正文。运行 `control-notes browser fill --role textbox --name "Title" --value "Release checklist"` 和 `control-notes browser fill --role textbox --name "Body" --value "Tag and publish"`。`Save note` 按钮变为可用。
- **保存笔记。** 选择 `Save note`。运行 `control-notes browser click --role button --name "Save note"`。一个名为 `Note saved` 的状态出现，标题显示为 `Release checklist`。
- **确认持久化。** 返回笔记列表并重新打开该笔记。运行 `control-notes browser click --role link --name "All notes"` 和 `control-notes browser click --role link --name "Release checklist"`。编辑器显示两个已保存的值。
- **取消草稿。** 打开一个新笔记，输入 `Discard me`，然后选择 `Cancel`。运行 `control-notes browser click --role button --name "New note"`、`control-notes browser fill --role textbox --name "Title" --value "Discard me"`，以及 `control-notes browser click --role button --name "Cancel"`。笔记列表返回，且不存在 `Discard me` 链接。
- **CLI 入口。** 创建第二条笔记。运行 `control-notes cli -- notes create --title "CLI note" --body "Created from terminal" --format json`。退出码 `0`，标准输出包含新笔记的 ID 和标题。
- **证明。** 从 `All notes` 重新打开两条已保存的笔记。运行 `control-notes browser snapshot --aria --path artifacts/create-note/list.aria.txt` 和 `control-notes browser screenshot --path artifacts/create-note/list.png`。这些产物应显示 `Release checklist` 和 `CLI note`。

## 注意事项

- 当文本框获得焦点时按 `n`，会输入字符，而不是打开新编辑器。
- 保存时标题会被去除首尾空白。应断言渲染后的标题，而不是草稿输入框的值。
- 仅有保存状态不足以作为证明。请从列表中重新打开笔记。
- 在夹具清理时删除 `Release checklist` 和 `CLI note`，但保留它们的证明产物。
