# Databricks 分析与系统表

## 此来源包含的内容

Databricks 是产品分析、数据管道和数仓遥测层。它与 Datadog 互为补充。Datadog 是*基础设施/运行时*视图，Databricks 是*产品/数据*视图（用户做了什么、运行了哪些实验、功能使用如何演变、某个阈值常量从何而来）。

- **产品分析事件。** `your_warehouse.events.analytics_track_event`（原始表），以及 `<your_analytics_db>.<schema>.<table>` 中经过类型化、去重的按事件划分的 dbt 模型。用户行为：功能调用、点击、接受/拒绝、提交、客户端上报的错误。
- **用量与计费事件。** `your_warehouse.events.usage_event` / `<your_analytics_db>.<schema>.stg_usage_events`，`your_warehouse.events.raw_model_event` / `<your_analytics_db>.<schema>.stg_raw_model_events`。用于由成本或用量驱动的决策。
- **实验 / 功能开关数据。** 曝光表和结果表。**Schema 因公司而异。** 在假设表名之前先用 `SHOW TABLES` 探查。
- **系统表。** `system.query.history`、`system.compute.warehouses`、`system.billing.*`、`system.access.audit`。用于回答"这条查询是否昂贵？"、"有人多久运行一次这条查询？"、"数仓负载在何时飙升？"
- **dbt 血缘。** `<your_analytics_db>.<schema>` 中的模型揭示了哪些管道依赖某张表或某个字段。上游变更经常会促成消费方代码的变更。
- **Databricks notebook。** 工程师在修改代码之前编写的探索性分析。**无法通过 SQL MCP 查询。** 如果你怀疑理由存在于某个 notebook 中，请将其标注为一项缺口。

## 如何搜索

使用 Databricks SQL MCP。主要工具：`execute_sql_read_only`。如果它返回 `statement_id`，请使用 `poll_sql_result` 轮询，而不是重新运行。

**查询前先摸清结构。** Schema 因公司而异。在信任某个表名之前先进行探查：

```sql
SHOW TABLES IN <your_analytics_db>.<schema> LIKE '*<keyword>*';
DESCRIBE TABLE <your_analytics_db>.<schema>.stg_<event>;
```

**每个查询都要限定时间范围。** 这些表非常庞大，不加约束的扫描会超时。对事件使用 `_timestamp` 过滤，对 `system.query.history` 使用 `start_time` 过滤，时间窗口应覆盖发布日期，通常为发布前后各约 30 天，只有在有充分理由时才扩大范围。

**优先使用类型化的 dbt 模型，而非原始表。** `<your_analytics_db>.<schema>.<table>` 经过去重、类型化，并采用了 liquid clustering。`your_warehouse.events.analytics_track_event` 存在重复数据，且 `properties_json` 未类型化。模型命名模式：`stg_<source>_<event_name_with_underscores>`，其中 `<source>` 为 `app`、`backend`、`website` 或 `cli`。当仅凭模式无法确定模型名时，请用 `SHOW TABLES` 确认确切的模型名。只有在不存在 dbt 模型，或你需要 dbt 刷新延迟期间的事件时，才改用原始表。

**类型化 dbt 模型上的列约定**（了解这些可以避免额外一次 `DESCRIBE` 往返）：

- `_timestamp`、`_id`、`_auth_id`、`_request_id`、`event_name`。每个模型都有的标准列
- `properties_<name>`。类型化的、下划线命名的事件属性（`properties_entrypoint`、`properties_size_bytes`，…）
- `context_team_id`、`context_client_version`、`context_country`、`context_client_os`。预先提取的客户端上下文

### 通常很有效的调查模式

根据目标选择匹配的表和列组合：

1. **事件使用轨迹。** 在 PR 合并前后 ±30 天的窗口内，统计相关 `stg_*` 模型的每日计数。如果在合并后一两天内，数量从零阶跃到稳定的量，这是 PR 推出了该功能的有力间接证据。如果数量衰减至零，则表明功能被弃用或删除。
2. **防御性检查的来源。** 在 PR *之前*的 14 天内，统计相关 `properties_<name>` 列的分布（中位数 / p99 / 最大值）。如果 p99 与目标的阈值常量吻合，则说明该数值可能是依据数据选定的。
3. **实验 / 功能开关查询。** 使用 `SHOW TABLES ... LIKE '*experiment*'` 找到曝光表，然后提取 PR 日期附近、相关开关键下按变体统计的曝光计数。
4. **为迁移、回填或性能重写查询历史证据。** 使用严格的 `start_time` 窗口，按 `statement_text ILIKE '%<table_or_symbol>%'` 过滤 `system.query.history`，可以找出可能促成该变更的昂贵查询（按 `total_duration_ms` 排序，或汇总 `SUM(read_bytes)`、`COUNT(*)`）。
5. **dbt 血缘。** 如果目标读取或写入某个 `<your_analytics_db>.<schema>` 模型，该模型自身的 git 历史（在本仓库中）通常包含其理由。应将此线索交还给 git 调查员，而不是自己去追查。

## 优质证据的样子

除上述模式之外：

- 某个错误分类事件的计数在防御性代码 PR 之后的几天内降至接近零。表明该 PR 解决了该类错误
- 某个曝光表行在 PR 上线日期前后，将目标的功能开关键标记为"已发布"/"已结束"的决定

## 常见误区

- **有记录 ≠ 由其导致。** 某个事件的存在只说明有人认为值得记录它，并不说明目标代码*因为*它而存在。在断定因果关系之前，请与 git 调查员提供的 PR/commit 引用配对验证。
- **静默的埋点变更。** 事件量的阶跃变化可能意味着开始记录了一个新事件，而非用户行为发生了改变。在把增长曲线解读为功能发布信号之前，请检查同一时间段内是否有埋点相关的 PR。
- **Schema 漂移。** 事件属性会不断演变。今天类型化 dbt 模型上的某一列，在目标代码编写时可能并不存在。较早的数据可能只在原始 `properties_json` 中携带该属性。
- **dbt 刷新延迟。** `<your_analytics_db>.<schema>.*` 按计划重建（通常为每小时或每天）。对于最近几小时的事件，请退回到 `your_warehouse.events.*`，并按 `_id` 去重。
- **公司特定的表。** 实验、功能开关、计费和用量表各不相同。在从未确认某张表是否存在的情况下报告其结果，是典型的失败模式。请先用 `SHOW TABLES` / `DESCRIBE TABLE` 探查。
- **保留期断崖。** 如果相关时间窗口早于该表的保留期或 dbt 模型的创建日期，这是*缺口*，而非空结果。请明确指出，以免综合者把"无结果"理解为"无活动"。
- **Notebook 不可查询。** SQL MCP 无法访问 Databricks notebook。如果你怀疑理由存在于某个 notebook 中，请将其作为缺口返回。

## 需要返回的内容

对于每一项相关发现：
- 类型（产品事件 / 实验曝光 / 用量或计费事件 / 系统表行 / dbt 模型）
- 完全限定的表名以及你运行的确切查询
- 所查询的时间窗口
- 紧凑的数值摘要（计数、百分位数、首次/最后出现的时间戳）。**不要转储原始行。**
- 与目标发布日期的时间相关性（例如："首行出现于 2024-08-15，PR #49074 合并于 2024-08-14"）
- 相关性与强度：直接 / 间接 / 弱
</content>
