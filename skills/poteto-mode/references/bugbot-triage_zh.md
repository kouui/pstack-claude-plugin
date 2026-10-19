# Bugbot 分诊

当 Babysit 手册（`../playbooks/babysit.md`）处理 Bugbot 或审查自动化的评论时，请使用本参考文档。目标不是默认忽略 Bugbot。目标是停止把每条评论都当作必须修改代码的要求。

## 决策规则

在采取行动之前，先对每个 Bugbot 线程进行分类：

- `fix`：评论指出了一个看似合理的正确性、安全、隐私、数据丢失、认证、计费、迁移、幂等性、竞态或已发布行为问题。在最底层的所属 PR 中修复它，然后回复提交 SHA 并解决该线程。
- `dismiss`：评论匹配一个已记录的低风险噪声模式，并且当前代码或上下文证明该顾虑不需要修改代码。回复一个简短的理由并解决该线程。
- `ask`：评论是新出现的、高严重性的、与安全/隐私/数据相关的，或含义不明确。请询问用户，不要猜测。

不确定时，就询问。跳过一条噪声较大的代码质量评论成本很低；跳过一个真实的数据或安全缺陷则不是。

## 已学习模式的格式

今后添加新模式时，请使用以下格式：

```markdown
### <short pattern name>

- Confidence: candidate | recurring | strong
- Skip when: <conditions that must be true>
- Do not skip when: <risk boundaries>
- Example signal: <phrases or code context that identify the pattern>
- Source: <PR/comment URL or short historical note>
```

对于只有一两个示例的情况，使用 `candidate`。在多次真实驳回之后，使用 `recurring`。只有当模式狭窄、经过反复验证且低风险时，才使用 `strong`。

## 可跳过的重复候选项

### 有意的 UI 或设计系统视觉变更

- Confidence: candidate
- Skip when: PR 描述、截图、设计评审或附近的代码已明确说明该视觉变更，并且 Bugbot 评论仅是在重申某个共享视觉默认值已经变化。
- Do not skip when: 评论涉及无障碍、焦点可见性、键盘导航、颜色对比度，或 PR 并未有意更改的组件 API 契约。
- Example signal: 关于焦点轮廓、按钮尺寸、间距或共享组件视觉默认值的评论，且负责人回复为 "intentional" 或 "intended"。

### 上层栈或栈内本地使用，Bugbot 无法看到

- Confidence: candidate
- Skip when: Bugbot 将某个导出、组件、辅助函数或文件标记为未使用，并且当前 forge 的 PR 列表和差异、上层栈的差异或 PR 上下文表明它被后续 PR 使用。
- Do not skip when: 当前 PR 不属于某个栈，该符号是公共 API，或无法验证所谓的上层使用。
- Example signal: "Exported component is never used"，且有人回复 "used upstack"。

### 并行实现期间的临时重复

- Confidence: candidate
- Skip when: PR 有意复制少量代码，以便在旧路径被删除、替换或验证之前，让新路径与旧路径并行存在。
- Do not skip when: 重复代码涉及安全、计费、数据访问、API 行为，或者一个长期共享的抽象明显能降低风险。
- Example signal: "Significant duplication" 或 "duplicated validation logic"，且负责人解释旧路径将被删除，或者重复逻辑是有意保留在本地的。

### 已有框架或组件不变式已覆盖该警告

- Confidence: candidate
- Skip when: 该顾虑已由共享组件、框架契约、类型不变式，或当前差异或附近代码中可见的单一事实来源所保证。
- Do not skip when: 该不变式是假定的但未被强制执行，依赖时序，或跨越异步/状态边界，在这些边界上数值可能发生分歧。
- Example signal: 关于内部 popover 缺少 max-height 的评论，而共享 popover 已强制执行视口边界；或者在本地检查的值与传入的值来自同一来源的可空值评论。

### 负责人声明的后续工作或延后清理

- Confidence: candidate
- Skip when: PR 负责人明确表示该问题是已知的后续工作，当前 PR 没有使该行为变差，并且评论不涉及高风险领域。
- Do not skip when: 代理在没有负责人输入的情况下行动，该问题是中等或高严重性的产品行为，或者延后处理会合并一个新的回归。
- Example signal: "I'll worry about that later" 或 "we'll delete this eventually"。

### 自行撤回或明确的误报规则评论

- Confidence: recurring
- Skip when: 评论正文或之后的 Bugbot 回复明确表示该发现已被撤回、合规或为误报，并且代理能够在本地验证相关规则。
- Do not skip when: 唯一的依据是某人在高风险问题上仅说 "false positive" 而没有解释。
- Example signal: 文件命名规则评论的正文说明该文件已经合规。

## 默认询问

即使之前的 PR 驳回过类似问题，也不要自动跳过以下类别：

- 安全、隐私、认证、计费、数据保留、训练数据和权限边界相关的发现。
- 高严重性发现。
- 迁移、模式、幂等性、并发和跨系统行为相关的发现。
- 建议的修复很小，并且明显能降低风险且不改变产品意图的评论。

历史数据显示，人类有时会驳回与安全或数据流相关的评论。应将这些视为负责人的判断事项，而不是团队范围内的跳过规则。

## 近期 Babysit 中的候选经验

在 babysit 期间或之后，如果新的经验对团队有用但尚未成熟，请追加到这里。在多个 PR 确认某模式之后，优先将重复出现的候选项提升到上面的章节中。

### 对原生浏览器行为的手动重新实现

- Confidence: candidate
- Skip when: 几乎从不。当差异用手动等价实现替换原生浏览器行为时（原生 sticky → 由 JS 定位的克隆，原生滚动目标 → 转发的 wheel/touch 事件，绘制顺序遮挡 → 遮罩/clip-path），Bugbot 针对此类代码的逻辑缺陷发现一直被证明是合理的。
- Do not skip when: 发现涉及事件转发缺口（wheel deltaMode、触摸平移、边缘滚动链、点击容差）、遮罩/裁剪命中测试偏差，或此类代码中的 observer 与 React 状态时序竞态。默认修复。
- Example signal: "masks do not affect hit-testing"、"overlay blocks wheel scroll"、"ignores deltaMode"、"runs in the IntersectionObserver callback before React applies state"。
- Source: 一个 sticky 遮挡 PR：六轮 Bugbot 检查，约十八项发现，每一项都被修复而非驳回。

### 契约测试漂移的声明可以低成本验证——先运行测试

- Confidence: candidate
- Skip when: 验证本身永远不要跳过；它只需一条命令。当 PR 附带一个固定协议或文档文字的契约测试（对 SKILL.md 的正则匹配、文档措辞的快照），并且 Bugbot 声称"测试与文档不再匹配"（或反之）时，在分类之前先在 PR 最新提交上运行该测试。失败的运行证实了该声明；通过的运行则是驳回回复的具体反证。
- Do not skip when: 不适用——这是验证捷径，而不是驳回模式。请注意，重复轮次中倾向于驳回的启发式规则在这里会误判：固定文字的测试之所以会漂移，正是因为早先的修复轮次修改了文字。
- Example signal: 在某个 PR 上出现 "Contract test omits the pre-fix wait"，而该 PR 早先的修复提交重写了被固定的段落；在最新提交上运行测试时，恰好在所引用的断言处失败。
- Source: 一个固定文字的 PR，共八轮 Bugbot 检查；尽管之前每一轮都已修复并解决，第 7 轮的声明仍然属实。

### 过期的安全审查发现已在同一 PR 后续中修复

- Confidence: candidate
- Skip when: 代理式安全审查（或类似工具）声称缺少授权/验证调用，而当前 PR 最新提交明确包含该确切的守卫（并附带测试），通常是在审查运行之后的后续加固提交中加入的。
- Do not skip when: 所引用的辅助函数对所讨论的主体是空操作，检查发生在它所保护的副作用之后，或者所声称的主体缺少覆盖。
- Example signal: 一个 HIGH 级别的 "missing authorization check" 发现，而在最新提交中，该确切守卫已在副作用之前被调用。
- Source: 一个 webhook 端点 PR，其加固提交晚于审查运行。

### 放宽刻意狭窄的错误条件会掩盖真正的错误

- Confidence: candidate
- Skip when: 该发现要求将一个狭窄的错误条件（特定的 `errno`、错误码或状态类别）扩大为兜底捕获，并且这种狭窄性编码了真实的区别。典型形式是依赖回退逻辑由 `ENOENT` 控制："二进制文件未安装"与"命令已运行但失败"是不同的情况。对任何非零退出码都重试，会将一次合法的失败（未找到、认证过期、网络错误）重新交给回退逻辑执行，然后报告回退逻辑的错误，从而隐藏真正的错误。
- Do not skip when: 狭窄条件遗漏了同一类别中的情况（例如同属"二进制文件不可用"的其他 errno，如 `EACCES`，或其他传输层失败），未处理的路径会丢失数据或留下部分状态，或者重试是幂等的，并且原始错误仍被抛出。
- Example signal: "only retries when X fails with ENOENT … never tries the fallback even when a working Y exists"，指向的代码中，回退逻辑是为缺失的依赖而设，而非为失败的操作而设。
- Source: 一个 CLI 重命名 PR，其回退逻辑是为缺失的二进制文件而存在，而非为失败的命令。
