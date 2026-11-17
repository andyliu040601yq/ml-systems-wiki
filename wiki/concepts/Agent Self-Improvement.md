# Agent Self-Improvement

**Sources:** [[BOXBOX]], [[Cache Engineering for Agents]]

In this wiki, **agent self-improvement** means changing some reusable part of an agent system based on task experience or evaluation feedback. The phrase covers several different targets. It does not by itself mean that a model updates its weights, that the agent improves without external evaluation, or that improvement generalizes to new tasks.

## Improvement targets

| Target | Example | What changes |
| --- | --- | --- |
| Harness and execution procedure | [[BOXBOX]] | The agent can revise guidance, tools, control flow, and state management during a task; a kernel validates and hot-reloads the new runtime. |
| Context or memory policy | [[Cache Engineering for Agents]] | A policy controls which KV states participate in future attention and is evaluated on held-out agent tasks; the serving implementation tracks operation history. |
| Model parameters | Not demonstrated by these sources | Would require a training or weight-update mechanism and evidence that resulting behavior improves. |

These mechanisms act at different layers and their reported gains cannot be directly compared. BOXBOX makes within-task changes to an executable harness; the cache-engineering paper reports a held-out recursively improved cache policy. Neither result alone establishes open-ended or general RSI.

## Questions for evaluating a self-improvement claim

- **What changes?** Separate model weights from prompts, tools, memory, harness code, and runtime policy.
- **What experience drives revision?** Identify whether the signal is task trajectory, test feedback, reference answers, or an external evaluator.
- **When does a revision take effect?** Distinguish offline training, between-task updates, between-attempt revisions, and hot reload during a live task.
- **What state and budget persist?** A new policy can appear better if it receives more time, resets context, retains useful artifacts, or is evaluated on different data.
- **How is improvement verified?** Look for held-out tasks, correctness gates, repeated runs, and protection against reward hacking or measurement leakage.
- **What is the transfer scope?** Within-task adaptation is not the same claim as cross-task generalization; a benchmark result is not general intelligence or autonomous research capability.

## Systems trade-offs and safeguards

Online revision can use task-specific evidence quickly, but interrupts work and may activate faulty or insecure code. Kernel-managed validation and rollback, as in [[BOXBOX]], address runtime continuity; the paper does not claim they form a complete security boundary. Offline revisions avoid changing a live runtime but can fail to transfer, as BOXBOX's PERFRACE evaluation illustrates for its particular Harness Evolution baseline. Evaluation should therefore account for both the quality of the revised procedure and the cost, correctness, safety, and continuity of the full execution process.

For harness revision strategies, see [[Online and Offline Harness Improvement]]. For the separate question of agent workflow metadata interacting with inference-engine state, see [[Harness–Engine Coordination]].
