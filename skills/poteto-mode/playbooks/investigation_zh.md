### 调查

**你对答案负责。规划、路由、撰写。**

调查类请求是只读的。它们产出带引用的解释或建议，而不是代码改动。

1. 通过 **how** skill 路由。对于动机类问题，还要通过 **why** skill 路由。
2. 吞吐量检查点只保留一行：`throughput checkpoint: n/a, read-only investigation`。
3. 产出 `how` 形式的输出（Overview / Key Concepts / How It Works / Where Things Live / Gotchas）；如果请求是在多个备选方案之间做决策，则产出带权衡表的建议。
4. 对回复应用 **unslop** skill。

不创建 PR，不执行 babysit，不使用 `architect`。除非调查是为了进行代码改动，此时要交还给用户，并重新路由到 Bug fix 或 Feature。

**回复：** 调查输出。对于"你确定吗？"这类问题，要给出你的真实判断及理由。如果前提有误，要提出反驳（参见 Autonomy）。
