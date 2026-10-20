# 已知问题清单

移植自 `cursor/plugins` 的 `pstack`(上游提交 `d457567`,版本 0.15.28)时发现的问题。先记录,移植完成后再分批处理。

## 说明

- 来源:10 个 haiku agent 并行通读上游文件后的报告。**这些条目尚未逐条对照原文核实**,处理前先读对应文件确认。
- 每个 `open` 条目对应一个 GitHub issue,标题以编号开头。`n/a` 条目没有建 issue。
- 状态:`open` 未处理,`wontfix` 决定不处理,`n/a` 移植时已不适用(例如相关文件没有移植),`done` 已处理。
- 类型:`upstream` 上游文档本身的缺陷,`port` 适配 Claude Code 带来的问题,`design` 设计层面的缺口。

## 上游内容缺陷

| ID | 类型 | 状态 | 问题 | 位置 |
| --- | --- | --- | --- | --- |
| P-01 ([#1](https://github.com/kouui/pstack-claude-plugin/issues/1)) | upstream | open | 正文基本为空,核心规则只在 description 里 | `skills/principle-exhaust-the-design-space/SKILL.md` |
| P-02 ([#2](https://github.com/kouui/pstack-claude-plugin/issues/2)) | upstream | open | 正文只剩一条"更新测试"的残句,核心规则只在 description 里 | `skills/principle-migrate-callers-then-delete-legacy-apis/SKILL.md` |
| P-03 ([#3](https://github.com/kouui/pstack-claude-plugin/issues/3)) | upstream | open | 同一文件里一处写 "Report only",另一处写 "I touch comments",是否直接改动注释不明确 | `agents/comment-sicko.md` |
| P-04 ([#4](https://github.com/kouui/pstack-claude-plugin/issues/4)) | upstream | open | 步骤编号缺 3 | `skills/swarm/SKILL.md` |
| P-05 ([#5](https://github.com/kouui/pstack-claude-plugin/issues/5)) | upstream | open | 恢复笔记写到 `/tmp/<slug>-resume.md`,与"临时文件放在会话 scratchpad"的约定冲突 | `skills/poteto-mode/playbooks/pause-safely.md` |
| P-06 ([#6](https://github.com/kouui/pstack-claude-plugin/issues/6)) | upstream | open | 用 BSD/macOS 的 `stat -f`、`date -r`,在 Linux 上 `LAST_CHAT` 等列可能不正确;还从 `~/.cursor/projects/` 读聊天记录 | `skills/poteto-mode/scripts/worktree-audit.sh` |
| P-07 ([#7](https://github.com/kouui/pstack-claude-plugin/issues/7)) | upstream | open | 稳定编号缺 1、2、4、6、21,确认是有意留空还是文件丢失 | `skills/unslop/SKILL.md` |
| P-08 | upstream | n/a | `setup-benny` 写"不要提交",`FOR_AGENTS.md` 要求启用前提交 `.cursor` 文件 | `automations/benny/`(未移植) |
| P-09 | upstream | n/a | `verify-existing-fix.md` 的 Confirmed 分支要回复源线程,而 reproduce 规定源线程最多一条主动回复 | `automations/benny/`(未移植) |
| P-10 | upstream | n/a | 配置里用 emoji 状态,SKILL.md 正文写"plain Unicode status strings" | `automations/benny/`(未移植) |

## 设计缺口

| ID | 类型 | 状态 | 问题 |
| --- | --- | --- | --- |
| D-01 ([#8](https://github.com/kouui/pstack-claude-plugin/issues/8)) | design | open | 决策日志(`show-me-your-work`)没有自动触发器,靠 agent 按指令手动调用;24 条 principle 里也没有一条直接规定要记录决策。可考虑用 hook 或结构化检查补上 |
| D-02 ([#9](https://github.com/kouui/pstack-claude-plugin/issues/9)) | design | open | `verify-<app>` 技能家族只被 `poteto-help` 的 recipe 引用,`poteto-mode` 的 playbook 没有直接引用它,验证优先靠 description 被发现。bug-fix、feature 等 playbook 应显式要求"有 verify skill 就用它" |
| D-03 ([#10](https://github.com/kouui/pstack-claude-plugin/issues/10)) | design | open | 验证 skill 的 feature map 没有 Claude Code 版的示例,示例里的 `control-notes` 是虚构 harness |
| D-04 ([#11](https://github.com/kouui/pstack-claude-plugin/issues/11)) | design | open | 编排台账(`orch`)只在 `orchestrate` playbook 中使用,其余长任务没有同样的 SHA 绑定验证记录 |

## 移植到 Claude Code 带来的问题

| ID | 类型 | 状态 | 问题 |
| --- | --- | --- | --- |
| C-01 ([#12](https://github.com/kouui/pstack-claude-plugin/issues/12)) | port | open | **模型**:上游默认值是 `grok-4.7-xhigh-fast`(代码)、`claude-opus-5-5-xhigh`(判断)等。本仓库先只支持 opus-5.5、sonnet-5.5、haiku-5.5,其余按需添加。涉及 `setup-pstack`、`poteto-mode`、`interrogate`、`arena`、`how`、`why`、`swarm`、`reflect` 等所有读 `pstack-models` 规则的 skill |
| C-02 ([#13](https://github.com/kouui/pstack-claude-plugin/issues/13)) | port | open | **适配层**:skill 正文里的 `Task`、`AskQuestion`、`generalPurpose`、`readonly`、`.cursor/skills`、`agent-transcripts`、速度/slug 等已改为直接写 Claude Code 的名字。`docs/harness/claude-code.md` 暂时保留作兜底,等全部 skill 复查后再精简或删除 |
| C-03 ([#14](https://github.com/kouui/pstack-claude-plugin/issues/14)) | port | done | **技能调用**:适配文档说 Skill 工具拒绝多数 pstack skill,要用 `Read` 读 SKILL.md。需在 Claude Code 里实测:带 `disable-model-invocation: true` 的 skill 能否被 `/pstack:<name>` 调用,skill 之间如何互相调用 |
| C-04 ([#15](https://github.com/kouui/pstack-claude-plugin/issues/15)) | port | open | `poteto-agent` 在 `~/.cursor/plugins/` 里搜 `poteto-mode/SKILL.md`,Claude Code 下应使用 `${CLAUDE_PLUGIN_ROOT}` |
| C-05 ([#16](https://github.com/kouui/pstack-claude-plugin/issues/16)) | port | open | 子代理能力差异:`readonly: true` 只能映射为 `Explore`(仍有 Bash,靠 brief 约束);`resume` 不支持;`environment: cloud` 只能映射为 worktree。只读约束需要更可靠的做法 |
| C-06 ([#17](https://github.com/kouui/pstack-claude-plugin/issues/17)) | port | open | 适配文档没有覆盖 `orchestrate`、`/loop`、Cursor Projects 在 Claude Code 里的对应方式 |
| C-07 ([#18](https://github.com/kouui/pstack-claude-plugin/issues/18)) | port | open | 依赖 Cursor 独有功能的 skill 和步骤:`make-bot-ui`(Grok Bot、Tailscale)、Bugbot 分诊、Cursor Automations、`/visualize`、Origin forge。(cloud agent、`cursor-team-kit` 的 `control-cli`/`control-ui` 已移除,并行工作改用本地 worktree;`/deslop` 已由本仓库自带;`create-skill` 视为 Claude Code 内置。)需逐个决定删除、替换或保留为可跳过 |
| C-08 ([#19](https://github.com/kouui/pstack-claude-plugin/issues/19)) | port | open | 脚本(`orch`、`watch-pr`、`check-plan.mjs`)依赖 bun/node,未在本环境运行过测试;`watch-pr` 依赖 `gh` |
| C-10 ([#20](https://github.com/kouui/pstack-claude-plugin/issues/20)) | port | open | 只剩 Claude 系列模型后,`interrogate`、`arena`、`show-me-your-work` 等"异族模型交叉评审"的设计前提(不同模型家族)不再成立,只剩 opus 和 sonnet 的差异。需决定如何保持评审独立性(不同上下文、不同 rubric、不同模型档位) |
| C-11 ([#21](https://github.com/kouui/pstack-claude-plugin/issues/21)) | port | open | 移植时把 `grok-4.7-xhigh-fast` 机械替换为 `sonnet xhigh`,把 `claude-opus-5-5-xhigh` 替换为 `opus xhigh`。各角色是否应该用 haiku(如 `how explorer`、`why investigators`)尚未评估;`effort` 值是否被当前 Agent 工具接受也未实测 |
| C-12 ([#22](https://github.com/kouui/pstack-claude-plugin/issues/22)) | port | open | README 和 `docs/guide` 是上游作者口吻,仍大量提到 Cursor、`/` 菜单、Option+Enter、`/loop`、`.cursor/skills/` 等;`README.md` 仍链接 benny 自动化包(未移植,链接指向上游 GitHub) |
| C-13 ([#23](https://github.com/kouui/pstack-claude-plugin/issues/23)) | port | open | 推理强度只用 `high` 和 `medium`(已替换默认值并把预算改为两档)。`poteto-mode` 的 "Match the model to the job" 仍提到 fast 模式和 slug 的 `-fast` 后缀,Claude Code 没有对应概念,需清理 |
| C-09 ([#24](https://github.com/kouui/pstack-claude-plugin/issues/24)) | port | open | 已改为 Claude Code 路径(`~/.claude/rules/`、`.claude/skills/`、`~/.claude/projects/<slug>/`),`worktree-audit.sh` 也已改读 `~/.claude/projects`。仍待检查:README 中作者口吻的 Cursor 描述、Automations/benny 包、`make-bot-ui` |

## 其他

- 上游 `docs/guide` 的 10 章是为 Cursor 写的,移植后保持原样,之后需要整体改写成 Claude Code 版本。
- 上游的 `scripts/generate-pstack-harness-trees.mjs` 和 `harness/pstack/` 生成树机制没有移植:本仓库直接维护 Claude Code 版本。
