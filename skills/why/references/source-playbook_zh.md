# 来源操作手册

why 技能会为每个可用的证据类别派生一名调查员，每人只阅读下面一份特定来源的操作手册。这些手册以常见 MCP 为具体示例。若要适配同类别中的其他 MCP，请自行调整。

| 类别 | 手册 | 手册所示示例 MCP |
|---|---|---|
| 源代码控制历史 | [`code-archaeology.md`](./sources/code-archaeology.md) | git、`gh` |
| Issue / ticket 跟踪 | [`linear.md`](./sources/linear.md) | Linear（可适配 Jira、GitHub Issues、Plane、Shortcut） |
| 长文档 | [`notion.md`](./sources/notion.md) | Notion（可适配 Confluence、Google Docs、Coda） |
| 实时团队聊天 | [`slack.md`](./sources/slack.md) | Slack（可适配 Discord、Microsoft Teams、Mattermost） |
| 基础设施可观测性 | [`datadog.md`](./sources/datadog.md) | Datadog（可适配 New Relic、Honeycomb、Grafana、Splunk） |
| 错误 / 异常跟踪 | [`sentry.md`](./sources/sentry.md) | Sentry（可适配 Rollbar、Bugsnag、Airbrake） |
| 产品分析数据仓库 | [`databricks.md`](./sources/databricks.md) | Databricks SQL（可适配 Snowflake、BigQuery、ClickHouse、dbt） |

横向通用：

- [`incident-postmortem.md`](./sources/incident-postmortem.md)。如果目标代码看起来有防御性逻辑（空值检查、重试、超时、速率限制、功能开关、出口防护、OOM 处理器），请添加此手册。
