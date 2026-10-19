### 编写或修改 skill

**你对 skill 的语气负责。**

1. 使用 **create-skill** skill（Cursor 内置的用于编写 SKILL.md 文件的 skill）。
2. 验证 skill：frontmatter 包含 `name` 和 `description`，引用的文件存在，跨 skill 链接可以解析。
3. 如果是结构性内容，编写测试用例。如果是主观内容，则跳过。
4. 执行 **Opening a PR**。

存疑时，删除。只保留会改变决策的文字。告诉它做什么，省略原因。只有在没有解释就难以理解规则时才加以说明。语气与范围相匹配。指向结构性来源（类型、README、配置），遵循 **encode-lessons-in-structure** 原则 skill。通过路径委托给其他 skill。不要重复陈述。如果你反复遇到某个工作流且尚未被记录，则提议新建一个 skill。

**回复：** skill 的摘要、关键设计决策、验证说明。
