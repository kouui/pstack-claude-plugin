# Sentry 错误历史

## 如何搜索

使用 Sentry MCP。

1. **定位。** 如果你不知道项目 slug 和组织：

   ```
   find_organizations
   find_projects
   ```

2. **搜索与目标相关的 issue。**

   ```
   search_issues (natural language, e.g., "errors in PaymentService timeout", "unhandled exceptions in uploadFile")
   ```

   根据目标处理的异常类名、其函数或类名、它检查的错误消息字符串以及其文件路径来构建查询。

3. **按 release 和时间窗口缩小范围。**

   ```
   search_issue_events (filter by release, time, environment, trace ID, tags)
   get_issue_tag_values (for an issue, see distribution across versions, users, environments)
   ```

   对于疑似 issue，检查 **first seen** 和 **last seen**（停止时间是否与目标的发布日期吻合）、**受影响的 release**（哪些版本出现过它，哪个版本是修复版本）以及**频率走势**（是否出现过峰值，之后是否已被解决）。

4. **拉取完整事件以获取上下文。**

   ```
   get_sentry_resource (pass a Sentry URL or type+ID)
   ```

   堆栈跟踪是否经过目标代码？tags 和 breadcrumbs 是否符合目标所防御的条件？

5. **检查目标附近发布的 release。**

   ```
   find_releases (around the commit date of the target)
   ```

   将 release 版本与 PR 的合并日期进行交叉比对。

6. **谨慎使用 Seer。**

   ```
   analyze_issue_with_seer
   ```

   Seer 的 AI 根因分析是假设生成器。将其视为推断，而非权威结论。实际事件和堆栈跟踪才是主要证据。

## 需要返回的内容

对于每个相关 issue：
- Issue ID 和标题
- 项目和组织
- First seen / last seen 时间戳
- 事件数量（如果已知，还包括采样率）
- 受影响的 release
- 一段有代表性的堆栈跟踪摘要，显示其与目标的相关性（逐字摘录，而非概述）
- First/last-seen 与目标发布日期的关联
- Issue 链接
- 任何作者评论或解决说明
