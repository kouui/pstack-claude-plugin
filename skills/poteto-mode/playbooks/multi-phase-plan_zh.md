### 多阶段或多 PR 计划

**你负责计划，而不是代码。计划是一份由负责人逐项执行的检查清单，操作员根据证据进行审计。** 计划本身就是交付物。不要实现。

1. 当改动只涉及一两个文件且方法明显时，跳过计划。说明这一点并停止。
2. 在编写之前，通过原型验证来解决未决问题。对每个问题运行 `playbooks/prototype.md`。保留分支、SHA 和截图，供附录 A 使用。只对无法通过任何运行解决的产品或偏好决策向操作员提问。给出选项（**never-block-on-the-human** 原则 skill）。
3. 使用 `subagent_type: "poteto-agent"` 在子代理中探索，并按子代理部分的规定为每个子代理指定明确的模型（**guard-the-context-window** 原则 skill）。根据任务为每个子代理选择推理级别以及快速或标准速度，依据子代理部分的规定。每个子代理返回文件指针、约定、测试命令和入口点。不要内联大段转储。
4. 将下面的骨架复制到计划文件中，并填写每个占位符。除非操作员指定路径，否则将文件写入 agent store 的 `docs/` 下。保留所示的每个标题和每个子块，并保持顺序。每个 PR 一个章节。一个 PR 就是一个具有自身证据的变更（**sequence-verifiable-units** 原则 skill）。在 **How to read this** 中指明执行 playbook。根据 `playbooks/autopilot-stack.md` 末尾的规则，在 `playbooks/autopilot-full.md` 和 `playbooks/autopilot-stack.md` 之间选择。常驻项目使用 `playbooks/orchestrate.md`。
5. 先按 `/technical-writing` 完整撰写，然后运行 `/unslop`。正文是一种 Diátaxis 模式，即操作指南。附录承载解释和参考。每个标题都要说明任务或发现。不使用长破折号。句中不使用冒号。
6. 运行 `node pstack/skills/poteto-mode/scripts/check-plan.mjs <plan.md>`，并修复它输出的每一行（**encode-lessons-in-structure** 原则 skill）。
7. 交还计划。发布计划路径和脚本输出，然后停止。执行在操作员明确批准后开始，并按计划指定的执行 playbook 进行。

**验证。** 仅有测试不足以作为验证。只有当某个 PR 的 unit、live 和 perf 检查框全部勾选时，该 PR 才算通过验证（**prove-it-works** 原则 skill）。这句话就是验证规则。每个验证块都以它开头。live 块是强制性的。在 PR head 上运行十个 lane，依据 **swarm** skill，通过项目自己的验证 skill（`verify-<app>`，见 `/create-verification-skill`）或项目自己的脚本驱动真实界面，使用 `swarm workers` 模型（默认 `sonnet high`）。每个 lane 是一个检查框，包含具体场景、它保存的截图以及通过判定条件。其中一个 lane 是 **针对 trunk 的回归 lane。** 它在 trunk 和 head 上运行同一个承载关键功能的场景。如果 trunk 没有该功能，该 lane 记录这一事实，并对该差异新增的行为以及用户等待的最终状态进行把关，而不是编造 trunk 的结果。性能门禁是双向的。trunk 和 head 都必须产生指定的指标。如果 trunk 缺少该功能，还要隔离该差异新增的工作，并为该工作以及用户等待的端到端状态设定绝对预算。不要在不可比的场景之间声称比率。性能块需指明指标、交错探测、先测得的 trunk 基线，以及导致失败的规则和数值。改变交互的 PR 需经过审查门禁。操作员在合并前通过聊天查看截图和视频进行审查。不改变交互的 PR 写入 `**Review gate.** None. <PR id> is not review-gated.`，并且其下不设检查框。

**控制 skill。** 按界面选择。浏览器、Electron、Web UI、CLI 和 TUI 使用项目自己的验证 skill（`verify-<app>`，见 `/create-verification-skill`）或项目自己的脚本来驱动。原生移动端使用仓库中已有的任何模拟器驱动 skill。触及两个界面的 PR 需要在两个界面上都设置 lane。没有控制 skill 的界面属于附录 C 中的风险，其 live 块仍需说明每个 lane 如何驱动它。

````markdown
# <Program> plan

<Under ten lines. What changes, for whom, the rule the program enforces, and the PR ids in order.>

## How to read this

One box is one unit of work. Every box names the evidence that checks it. A nested box is a sub-step of the box above it. Check a box only when its evidence exists, a file, a log line, a screenshot, a test run, or a SHA. The body is a how-to. The appendices explain and record.

The program runs `pstack/skills/poteto-mode/playbooks/<execution playbook>.md`. <Who merges, and which PR ids are the operator's items that stop at merge-ready.>

Tests alone are not sufficient verification. A PR is verified only when its unit, live, and perf boxes are all checked.

## Program checklist

### Arm the program

- [ ] State the protocol and this plan to the operator, then stop. Start execution only on the operator's explicit go.
- [ ] Read these from trunk at program start. Re-read them at every tick.
  - [ ] `git show origin/main:pstack/skills/poteto-mode/playbooks/<execution playbook>.md`
  - [ ] `git show origin/main:pstack/skills/swarm/SKILL.md`
  - [ ] `git show origin/main:<control skill path>`
  - [ ] `git show origin/main:pstack/skills/poteto-mode/playbooks/opening-a-pr.md`
  - [ ] `git show origin/main:pstack/skills/<each other leaf skill the program uses>`
- [ ] On the operator's go, arm the audit tick as `/loop 1h` with the tick prompt below. Never leave the cadence to memory.
- [ ] Use this tick prompt, verbatim. "Re-read the execution playbook from trunk. Audit the operation against it and fix drift in this tick. Probe every active lane and judge progress by side effects only. Stand down a stuck lane and dispatch its replacement now. Then post a short status message to the operator in chat only when the audit found a tracked change that no earlier status message reported, such as a PR opened, a code-ready head, a round launched or closed, a verdict, a merge, a stuck agent and the action taken, a blocker added or cleared, or a decision only the operator can make. Name every such change and nothing else. Do not repeat a table, the merged list, or an unchanged blocker. If the audit found none, end the turn with no reply text. Either way, log this tick's row in your decision trail. The row names the items reported, or none."
- [ ] On the operator's hold or stand-down, send every owner a zero-writes order at once.

### Spawn owners

- [ ] Spawn one owner per PR with the full lifecycle the execution playbook names.
- [ ] Follow this dependency graph. Start dependent work only after its parent merges, or base it on the parent branch when the execution playbook stacks.
  - [ ] <PR id> and <PR id> are independent and first. Both branch from `main`.
  - [ ] <PR id> after <PR id>.
- [ ] Hold the file boundaries. <PR id or class> touches only `<glob>`.
- [ ] Hold the review gate. <PR ids> change an interaction. They wait for the operator's review in chat with screenshots and a video before merge.

### PR mechanics, for every PR

- [ ] Resolve the forge once. Default to `gh`; if `command -v origin` succeeds and Origin can resolve the repository, use `origin pr` for every PR operation. Record any fallback to `gh`. Never require `gt`.
- [ ] Open the PR ready, never draft, per **Opening a PR**. Use the run's built-in PR tool when it has one, else `origin pr create --status open --base <base-branch>` or `gh pr create --base <base-branch>` according to the resolved forge. A stack child targets its parent branch.
- [ ] Run the repo's lint and typecheck once before the PR-facing push. Push with hooks on.
- [ ] Run `/deslop` before each commit and `/no-comments` before review.
- [ ] Triage every Bugbot and security-reviewer comment per `../references/bugbot-triage.md`.
- [ ] Rebase onto current trunk before the code-ready report and babysit. Keep that merge base in fix rounds. Rebase again only at merge prep, on a `git merge-tree` conflict with trunk, or on a CI failure that comes from a change on trunk.

### Verdict and merge, for every PR

- [ ] A full swarm runs once, at the code-ready head. Another full swarm runs only for new behavior, a redesign, or a large new diff. A rebase, a merge-conflict resolution, or a minor fix gets a single verifier at the new head. Run each full swarm per `pstack/skills/swarm/SKILL.md`. One gates lane. The ten live lanes from the PR's **Verify, live** block. The perf lane from its **Verify, perf** block. Two or more audit lanes, each with its own focus, that read the diff and the receipts and distrust the PR body. The root audits the receipts in the merge-ready report before the verdict.
- [ ] Clean only when every lane is `PASS`. Findings go back to the owner, including a defect that a lane filed as a note. A new head gets a fresh verdict, except for results that stay valid under the patch-id rule in `playbooks/shipping.md`.
- [ ] <The merge or append rule from the execution playbook, with the patch-id rule from `playbooks/shipping.md`.>

### Boot recipe, for every live lane

Each live lane runs in its own local git worktree at the PR head. Drive it with the project's own verification skill (`verify-<app>`, see `/create-verification-skill`) or the project's own scripts.

- [ ] `git fetch origin <head-branch> && git checkout <head SHA>`.
- [ ] <Start the backend and the surface. Wait for ready.>
- [ ] <Deliver input only through the control skill's commands. Name the read-only diagnostics.>
- [ ] Save every screenshot to `/tmp/swarm-<pr-id>/worker-<n>/<slug>.png` and return the paths with the report.

## <Task as a verb phrase> (<PR id>)

**Depends on.** <PR id, or None.>

**Files.**

- [ ] Edit `<path>`.
- [ ] Create `<path>`.
- [ ] Delete `<path>`.

**Build.**

- [ ] <One change. Name the symbol and the file.>

**You see.**

- [ ] <One observable result, with the exact log line or screen state.>

**Verify, unit.** Tests alone are not sufficient verification. A PR is verified only when its unit, live, and perf boxes are all checked.

- [ ] <Test file and the case it gains.> Run `<command>`.

**Verify, live.** Tests alone are not sufficient verification. A PR is verified only when its unit, live, and perf boxes are all checked. Ten lanes on `<swarm workers model>` at the PR head, per the boot recipe.

- [ ] Lane 1. Regression lane against trunk. Run <the same load-bearing scenario> at trunk and head. If trunk lacks the feature, record that and gate <the behavior the diff adds plus the end state the user waits for>. Save `<slug>.png`. Pass when <predicate>.
- [ ] Lane 2. <Scenario.> Save `<slug>.png`. Pass when <predicate>.
- [ ] Lane 3. <Scenario.> Save `<slug>.png`. Pass when <predicate>.
- [ ] Lane 4. <Scenario.> Save `<slug>.png`. Pass when <predicate>.
- [ ] Lane 5. <Scenario.> Save `<slug>.png`. Pass when <predicate>.
- [ ] Lane 6. <Scenario.> Save `<slug>.png`. Pass when <predicate>.
- [ ] Lane 7. <Scenario.> Save `<slug>.png`. Pass when <predicate>.
- [ ] Lane 8. <Scenario.> Save `<slug>.png`. Pass when <predicate>.
- [ ] Lane 9. <Scenario.> Save `<slug>.png`. Pass when <predicate>.
- [ ] Lane 10. <Scenario.> Save `<slug>.png`. Pass when <predicate>.

**Verify, perf.** Tests alone are not sufficient verification. A PR is verified only when its unit, live, and perf boxes are all checked.

- [ ] Metric. <What is measured at both trunk and head. If trunk lacks the feature, also name the diff-added work and the end-to-end state the user waits for.>
- [ ] Probe. <The command or procedure, run at trunk and at the head, interleaved. Both sides must produce the metric.>
- [ ] Baseline. Record the trunk <value> first.
- [ ] Rule. <Head against trunk, with the number that fails. If the scenarios differ, add absolute budgets for the diff-added work and the user-visible end state instead of an invalid ratio.>

**Review gate.** The operator reviews before merge.

- [ ] Copy lane <n> screenshots into `<media path>/<pr-id>-review-<slug>.png`.
- [ ] Record a 30 to 60 second video of the change from a lane's worktree. Save it as `<media path>/<pr-id>-review.mp4`.
- [ ] Post the screenshots and the video in chat. Stop at merge-ready. Wait for the operator's click.

**Merge.**

- [ ] Root's clean verdict at the exact head SHA.
- [ ] Bugbot triage done.
- [ ] Rebased onto current trunk after the verdict, patch-id unchanged.
- [ ] <The owner squash-merges its own PR, or the root appends it to the base-branch stack and the operator lands it bottom-up.>

## Close the program

- [ ] Every box above is checked with its evidence.
- [ ] Reply to the operator with the report the execution playbook names.

## Appendix A. Prototype evidence

<Each open question a prototype answered, with the branch, the SHA, and the artifact links. Each question that stays unproven.>

## Appendix B. Alternatives rejected

<Each approach weighed and why it lost.>

## Appendix C. Risks

<Each risk with the PR it lands in and what the owner watches.>

## Appendix D. Links and reading list

<Docs to read before editing. Which PRs get `pstack/skills/how/SKILL.md` and `pstack/skills/interrogate/SKILL.md`. The trail per `pstack/skills/show-me-your-work/SKILL.md`.>
````

**回复：** 计划路径、PR id 及其依赖关系以及需审查门禁的集合、原型证明了什么以及什么仍未证明，以及检查脚本的输出。
