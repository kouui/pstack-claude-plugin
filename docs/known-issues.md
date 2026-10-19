# 已知问题清单

移植自 `cursor/plugins` 的 `pstack`(上游提交 `d457567`,版本 0.15.28)时发现的问题。先记录,移植完成后再分批处理。

## 说明

- 来源:10 个 haiku agent 并行通读上游文件后的报告。**这些条目尚未逐条对照原文核实**,处理前先读对应文件确认。
- 状态:`open` 未处理,`wontfix` 决定不处理,`n/a` 移植时已不适用(例如相关文件没有移植),`done` 已处理。
- 类型:`upstream` 上游文档本身的缺陷,`port` 适配 Claude Code 带来的问题,`design` 设计层面的缺口。

## 上游内容缺陷

| ID | 类型 | 状态 | 问题 | 位置 |
| --- | --- | --- | --- | --- |
| P-01 | upstream | open | 正文基本为空,核心规则只在 description 里 | `skills/principle-exhaust-the-design-space/SKILL.md` |
| P-02 | upstream | open | 正文只剩一条"更新测试"的残句,核心规则只在 description 里 | `skills/principle-migrate-callers-then-delete-legacy-apis/SKILL.md` |
| P-03 | upstream | open | 同一文件里一处写 "Report only",另一处写 "I touch comments",是否直接改动注释不明确 | `agents/comment-sicko.md` |
| P-04 | upstream | open | 步骤编号缺 3 | `skills/swarm/SKILL.md` |
| P-05 | upstream | open | 恢复笔记写到 `/tmp/<slug>-resume.md`,与"临时文件放在会话 scratchpad"的约定冲突 | `skills/poteto-mode/playbooks/pause-safely.md` |
| P-06 | upstream | open | 用 BSD/macOS 的 `stat -f`、`date -r`,在 Linux 上 `LAST_CHAT` 等列可能不正确;还从 `~/.cursor/projects/` 读聊天记录 | `skills/poteto-mode/scripts/worktree-audit.sh` |
| P-07 | upstream | open | 稳定编号缺 1、2、4、6、21,确认是有意留空还是文件丢失 | `skills/unslop/SKILL.md` |
| P-08 | upstream | n/a | `setup-benny` 写"不要提交",`FOR_AGENTS.md` 要求启用前提交 `.cursor` 文件 | `automations/benny/`(未移植) |
| P-09 | upstream | n/a | `verify-existing-fix.md` 的 Confirmed 分支要回复源线程,而 reproduce 规定源线程最多一条主动回复 | `automations/benny/`(未移植) |
| P-10 | upstream | n/a | 配置里用 emoji 状态,SKILL.md 正文写"plain Unicode status strings" | `automations/benny/`(未移植) |

## 设计缺口

| ID | 类型 | 状态 | 问题 |
| --- | --- | --- | --- |
| D-01 | design | open | 决策日志(`show-me-your-work`)没有自动触发器,靠 agent 按指令手动调用;24 条 principle 里也没有一条直接规定要记录决策。可考虑用 hook 或结构化检查补上 |
| D-02 | design | open | `verify-<app>` 技能家族只被 `poteto-help` 的 recipe 引用,`poteto-mode` 的 playbook 没有直接引用它,验证优先靠 description 被发现。bug-fix、feature 等 playbook 应显式要求"有 verify skill 就用它" |
| D-03 | design | open | 验证 skill 的 feature map 没有 Claude Code 版的示例,示例里的 `control-notes` 是虚构 harness |
| D-04 | design | open | 编排台账(`orch`)只在 `orchestrate` playbook 中使用,其余长任务没有同样的 SHA 绑定验证记录 |

## 移植到 Claude Code 带来的问题

| ID | 类型 | 状态 | 问题 |
| --- | --- | --- | --- |
| C-01 | port | open | **模型**:上游默认值是 `grok-4.7-xhigh-fast`(代码)、`claude-opus-5-5-xhigh`(判断)等。本仓库先只支持 opus-5.5、sonnet-5.5、haiku-5.5,其余按需添加。涉及 `setup-pstack`、`poteto-mode`、`interrogate`、`arena`、`how`、`why`、`swarm`、`reflect` 等所有读 `pstack-models` 规则的 skill |
| C-02 | port | open | **适配层**:skill 正文仍使用 Cursor 的名字(`Task`、`AskQuestion`、`generalPurpose`、`readonly`、`.cursor/skills`、`agent-transcripts`),靠 `docs/harness/claude-code.md` 每会话读一次来翻译。应改为直接写 Claude Code 的名字,并去掉适配文档 |
| C-03 | port | open | **技能调用**:适配文档说 Skill 工具拒绝多数 pstack skill,要用 `Read` 读 SKILL.md。需在 Claude Code 里实测:带 `disable-model-invocation: true` 的 skill 能否被 `/pstack:<name>` 调用,skill 之间如何互相调用 |
| C-04 | port | open | `poteto-agent` 在 `~/.cursor/plugins/` 里搜 `poteto-mode/SKILL.md`,Claude Code 下应使用 `${CLAUDE_PLUGIN_ROOT}` |
| C-05 | port | open | 子代理能力差异:`readonly: true` 只能映射为 `Explore`(仍有 Bash,靠 brief 约束);`resume` 不支持;`environment: cloud` 只能映射为 worktree。只读约束需要更可靠的做法 |
| C-06 | port | open | 适配文档没有覆盖 `orchestrate`、`/loop`、Cursor Projects 在 Claude Code 里的对应方式 |
| C-07 | port | open | 依赖 Cursor 独有功能的 skill 和步骤:`make-bot-ui`(Grok Bot、Tailscale)、Bugbot 分诊、Cursor Automations、cloud agent、`/visualize`、内置 `create-skill`、cursor-team-kit 的 `/deslop`、Origin forge。需逐个决定删除、替换或保留为可跳过 |
| C-08 | port | open | 脚本(`orch`、`watch-pr`、`check-plan.mjs`)依赖 bun/node,未在本环境运行过测试;`watch-pr` 依赖 `gh` |
| C-09 | port | open | 多个 skill 的文档示例写死 Cursor 路径(`~/.cursor/rules/pstack-models.mdc`、`.cursor/skills/verify-<app>/`、`~/.cursor/projects/<slug>/agent-transcripts`) |

## 其他

- 上游 `docs/guide` 的 10 章是为 Cursor 写的,移植后保持原样,之后需要整体改写成 Claude Code 版本。
- 上游的 `scripts/generate-pstack-harness-trees.mjs` 和 `harness/pstack/` 生成树机制没有移植:本仓库直接维护 Claude Code 版本。
