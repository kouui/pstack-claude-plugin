# Datadog 遥测

## 此来源包含的内容

Datadog 保存运行时记录，即生产环境中实际发生的事情，而不是计划或讨论过的内容。

- **指标。** 团队埋点的计数器、仪表和直方图。指标的*存在*本身就是证据，说明有人认为这个数值值得监控。
- **监视器与告警。** 团队判定值得叫醒某人的条件。监视器因 `rate_limit_hit > 10/min` 而触发，是团队担心该阈值的直接证据。
- **仪表盘。** 团队为某个子系统认为重要的内容所整理的视图。
- **APM 追踪与 span。** 请求级别的运行时数据，用于回答“为什么这么慢”或“这里为什么会超时”之类的问题。
- **日志。** 高容量的事件记录，通常包含促使防御性代码产生的错误情况。
- **事件。** 带有时间线和关联复盘报告的正式事件记录。
- **笔记本。** 团队的探索性调查，通常包含假设和分析。

## 如何搜索

使用 Datadog MCP。先广泛搜索，再逐步缩小范围。

1. **确定负责的服务。**

   ```
   search_datadog_services (filter by name or team)
   search_datadog_service_dependencies (see upstream/downstream)
   ```

2. **优先查看仪表盘和监视器。**

   ```
   search_datadog_dashboards (query: feature name, service name, symbol)
   search_datadog_monitors   (same queries)
   ```

   当某个仪表盘或监视器覆盖了目标时，记录它的查询和监控的阈值。阈值常常就是“为什么这里被限制在 N？”这一问题的答案。

3. **目标周围的指标。**

   ```
   search_datadog_metrics (by name pattern, e.g., the feature or symbol)
   get_datadog_metric_context (metadata: description, units, tags)
   get_datadog_metric (timeseries; "was there a spike around the PR date?")
   ```

   将指标的变化趋势与目标的添加或修改日期进行关联，是很有力的佐证：“`payment_timeout` 指标在 2023-11-03 出现峰值，而重试逻辑在 2023-11-06 合并。”

4. **日志。窄查询，不要全部导出。**

   ```
   search_datadog_logs (raw log patterns near the target, set use_log_patterns=true)
   analyze_datadog_logs (SQL-style aggregations, only when you need counts)
   ```

   使用符号、错误字符串或功能名称进行搜索。**强烈建议使用限定时间范围的查询**（例如变更前后 30 天）。

5. **APM span 与追踪。**

   ```
   aggregate_spans    (stats: "how often does this endpoint fail?")
   search_datadog_spans (inspect individual spans)
   get_datadog_trace  (a specific trace ID)
   ```

   适用于超时、重试、慢路径以及跨服务行为的分析。

6. **事件。**

   ```
   search_datadog_incidents (by title, team, date range)
   get_datadog_incident     (full detail for a specific incident)
   ```

   如果目标看起来是防御性代码，请搜索其添加时间前后的事件。时间线中包含“为 X 添加了防御性检查”的事件，是近乎直接的证据。

## 好的证据是什么样的

- 监视器的查询和阈值与代码强制执行的约束相匹配（代码将值限制在 100，监视器在请求数超过 100/min 时告警）
- 由目标的作者创建的仪表盘，其组件对应代码所测量或防范的内容
- 指标显示在代码合并前出现生产峰值，之后数值趋于稳定
- 事件记录引用了目标代码、相同的符号或相同的错误字符串
- 日志显示防御性代码所能预防的特定错误模式，且时间戳位于变更之前的时间窗口内

## 常见误区

- **相关不等于因果。** PR 之前出现峰值、之后趋于稳定，这只能说明一种可能，并不确定，因为同一时间窗口内可能还合并了其他变更。请检查相邻的 PR。
- **过度解读你找到的图表。** 图表反映的是制作者的视角。名为“重试成功率”的图表只能证明团队关心重试成功率，并不能证明某行特定代码为何存在。
- **遥测数据消失。** 指标可能被重命名、删除，或保留期很短。如果找不到相关时间窗口内的数据，这属于数据缺失，而不是否定的结果。
- **大规模的噪声。** 请按服务、标签和时间窗口积极地收窄日志搜索范围。使用 `analyze_datadog_logs` 进行聚合，而不是直接导出原始日志。
- **被埋点不等于被引起。** 指标只能说明有人在意某件事并对其进行了测量，并不能说明代码是*因为*它而添加的。请与提交或 PR 的日期交叉核对。

## 需要返回的内容

对于每个相关项目：
- 类型（仪表盘 / 监视器 / 指标 / 日志模式 / 追踪 / 事件 / 笔记本）
- 标题或名称
- 链接或标识符（仪表盘 ID、监视器 ID、指标名称、事件 ID）
- 所有者/作者以及创建和修改日期
- 与问题相关的具体条件、查询或引文（尽可能逐字引用）
- 相关性：这说明了目标代码的什么信息，以及关联的强度如何
