# 事故与复盘上下文

- **Linear**：查找标签为 `incident`、`sev-*`、`postmortem-action-item`、`reliability` 的工单
- **Slack**：在目标代码添加日期前后，搜索 `#sev-*` 和 `#incident-*` 频道
- **Git**：提交信息类似 "fix for incident"、"add defensive check"、"revert"，且随后出现 "re-apply with..." 的提交，是强烈信号
