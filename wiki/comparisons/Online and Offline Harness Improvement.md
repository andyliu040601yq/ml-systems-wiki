# Online and Offline Harness Improvement

**Papers:** [[BOXBOX]], [[Cache Engineering for Agents]]

Harness and agent-policy improvement differs along two axes: whether changes are learned from reusable prior tasks or the current task, and whether they take effect before a task, between attempts, or inside an ongoing execution. BOXBOX evaluates one online point in this design space; the comparison below is grounded in its reported baselines and should not be read as a universal ranking.

| Approach | Feedback and timing | Execution continuity | Evidence in BOXBOX |
| --- | --- | --- | --- |
| Frozen harness | No revision during the task | One task-solving run | 1.442× Speedup@4h; $23.22 mean model cost/run |
| Offline Harness Evolution | Revises on training tasks, selects a harness on validation, then fixes it for test tasks | Test task uses the selected harness without in-task revision | 1.281×; $24.56/run. Offline costs bring the paper's estimate to at least $33.73 per test run when amortized as described. |
| Harness Scaling | External controller revises between attempts on the same task | Retains project progress but starts a new agent session, resetting execution context | 1.539×; $12.98/run |
| BOXBOX | Task-solving agent triggers revision from its current trajectory | Kernel hot-reloads after validation while preserving trajectory, workspace, and terminal session | 1.702×; $16.43/run |

All four rows use the paper's PERFRACE evaluation and matched four-hour budgets; BOXBOX is higher on Speedup@4h and the reported Speedup/$, while Harness Scaling has lower mean model cost. The trade-off matters: optimizing for model cost alone does not select the best speedup result, and aggregate measurements remain specific to the benchmark.

## Mechanistic differences

- **Who chooses to revise?** BOXBOX lets the task agent decide from emerging task evidence; Harness Scaling delegates revision timing to an outer controller.
- **What scope is editable?** BOXBOX exposes the full executable harness. The paper's restricted-editing ablation performs worse on its 14-task subset, but does not prove unrestricted editing is best in other settings.
- **What survives the update?** BOXBOX transfers the trajectory, terminal session, and workspace into the reloaded runtime. Harness Scaling preserves project files but resets conversational execution context between attempts.
- **What does “improvement” refer to?** A better harness can improve how the agent investigates and measures the project; it is distinct from directly editing the target project or updating model weights.

[[Cache Engineering for Agents]] offers a related but separate RSI example: its recursively improved policy changes active KV-cache participation and is evaluated on held-out tasks, rather than hot-reloading a harness in one live task. See [[Agent Self-Improvement]] for a broader taxonomy. [[Harness–Engine Coordination]] and [[Agentic Serving Scheduling HEAR and TAO]] address serving interfaces and scheduling, not harness self-revision.
