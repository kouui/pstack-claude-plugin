---
name: reflect
description: 对当前会话记录启动三个并行的审查子代理，提炼经验教训，并将每条经验路由为对现有 skill 的具体编辑。当用户说 reflect 时使用。
disable-model-invocation: true
---

# Reflect

在 Claude Code 中，每个会话开始时先阅读一次 [Claude Code 适配器](../../docs/harness/claude-code.md)，然后再遵循本 skill。它将 pstack skills 中的 Cursor 名称映射到 Claude Code。

挖掘当前对话中可持久保存的经验教训，然后将其归入 skill 编辑。

当用户说 "reflect" 或 "/reflect" 时调用。如果对话很琐碎、跑题，或已被父代理正确遵循的现有 skill 所覆盖，则跳过。一次性的事情不是经验教训。

## 1. 定位当前会话记录

在启动扇出之前，先在系统提示所指定的 `agent-transcripts/` 目录中找到本次对话的会话记录。不要跨 `~/.cursor/projects/*/` 进行 glob 搜索。那样会跨越工作区边界，并读取无关项目的私有聊天内容。

```bash
ls -t <agent-transcripts>/*.jsonl <agent-transcripts>/*/*.jsonl <agent-transcripts>/*/subagents/*.jsonl 2>/dev/null | head -10
```

这涵盖旧版扁平布局（`<id>.jsonl`）、当前嵌套布局（`<id>/<id>.jsonl`）以及子代理布局（`<parent>/subagents/<child>.jsonl`）。对每个候选文件，读取第一行 JSONL。取 `message.content[0].text` 包含对话开头用户提示的那个文件。如果没有匹配项，则改为传入一份精简的会话摘要。

## 2. 并行启动三个审查者

在同一条消息中发起三个 `Task` 调用，`subagent_type: generalPurpose`，agent 模式（`readonly: false`）。审查者需要 MCP 访问权限，以查询会话记录所引用的工单、聊天线程和可观测性追踪，而 readonly 会去除 MCP。

每个审查者和综合者都在 `pstack-models.md` 规则中有一行角色说明和一个默认值。将 `model` 设为该行的值；如果规则或该行缺失，则设为默认值。当值为 `auto` 或 `inherit-parent` 时，不设置 `model`。如果 Task 工具拒绝某个 slug，则使用默认值并说明这一点。如果它连默认值也拒绝，则使用其错误信息中同一系列中最接近的有效 slug。根据 **poteto-mode** skill 的 Subagents 部分，为每个子代理选择合适的推理级别以及快速或标准速度。

| 视角 | 角色行 | 默认 `model` | 提示模板 |
|---|---|---|---|
| Judgment | `reflect judgment, divergent, synthesizer` | `opus high` | `references/judgment-reviewer.md` |
| Tooling | `reflect tooling` | `sonnet high` | `references/tooling-reviewer.md` |
| Divergent | `reflect judgment, divergent, synthesizer` | `opus high` | `references/divergent-reviewer.md` |

将每个模板原样传入，并在标记处替换为会话记录路径或摘要。审查者在 `Task` 响应正文中返回发现。

## 3. 综合

发起一个 `Task` 调用，`subagent_type: generalPurpose`，`model` 取自 `reflect judgment, divergent, synthesizer` 行（默认 `opus high`），agent 模式（`readonly: false`）。它通过 MCP 抽查引用，而 readonly 会去除 MCP。原样传入 `references/synthesizer.md`，并在标记处内联每个审查者的完整输出。它返回 Accepted / Rejected / Backlog 列表。

## 4. 结构化强制检查

将任何 Accepted 项中，如果由 lint 规则、脚本、元数据标志或运行时检查来强制执行会更可靠的，移至 Backlog。参见 **encode-lessons-in-structure** 原则 skill。

## 5. 应用

在应用任何 Accepted 编辑之前，先展示综合者完整的 Accepted / Rejected / Backlog 输出，并等待明确批准。用户选择子集，并可以改变路由。Skill 变更会影响组织内所有未来的代理。不要自动应用。

无需等待，即可将每个 Backlog 项提交到团队的 devex 或 backlog 跟踪器。只有 Accepted 列表需要等待批准。

严格按照每个已批准行的 Routing 执行：

- 对现有 skill 的琐碎编辑（一行条目、精简的句子、更正的过时事实）：由父代理直接完成。
- 对现有 skill 的实质性编辑（新增章节、新的模式表、超过约 10 行）：交给 Cursor 内置的 `create-skill` skill，并运行其草拟 / 测试 / 迭代循环。
- `tune description: <skill path>`（skill 存在但在应当触发时未触发）：交给 `create-skill`，并运行其描述优化循环。
- `new skill via create-skill: <kebab-name>`：将创建工作交给 `create-skill`。不要临时自行设计结构。

如果你的环境提供 SKILL.md 验证器，在宣告完成之前，对每个被修改的 skill 运行它。

## 6. 向用户汇总

简短列表，无前言：

- 已应用的编辑：`<skill path>`。改了什么，每项一行。
- 新建的 skill：`<skill path>`。每项一行（很少见）。
- 提交到 devex 跟踪器的 Backlog：`<issue title>`（`<tags>`）。每项一行。
- 已丢弃：每条被拒绝的发现一行，附综合者给出的原因。
