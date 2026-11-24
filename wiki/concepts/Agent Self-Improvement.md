# Agent Self-Improvement

**Sources:** [[AIDE2]], [[Continual Harness]], [[BOXBOX]], [[Cache Engineering for Agents]]

In this wiki, **agent self-improvement** means changing some reusable part of an agent system based on task experience or evaluation feedback. The phrase covers several different targets. It does not by itself mean that a model updates its weights, that the agent improves without external evaluation, or that improvement generalizes to new tasks.

## Improvement targets

| Target | Example | What changes |
| --- | --- | --- |
| Harness and execution procedure | [[BOXBOX]] | The agent can revise guidance, tools, control flow, and state management during a task; a kernel validates and hot-reloads the new runtime. |
| Context or memory policy | [[Cache Engineering for Agents]] | A policy controls which KV states participate in future attention and is evaluated on held-out agent tasks; the serving implementation tracks operation history. |
| Model parameters, coupled with a changing harness | [[Continual Harness]] | A frontier teacher relabels low-reward student rollout windows, soft SFT updates the model between iterations, and the harness continues to refine online in a reset-free embodied task. |
| The research optimizer itself | [[AIDE2]] | An outer loop rewrites the research-agent code and retains candidates whose inner-loop optimization performance improves on private held-out tasks at fixed per-task budgets. |

These mechanisms act at different layers and their reported gains cannot be directly compared. BOXBOX makes within-task changes to an executable harness. Continual Harness combines that kind of in-episode harness adaptation with model-weight updates between iterations, but relies on a frontier teacher and is evaluated in Pokémon environments. AIDE2 adds an outer loop that improves the research agent's own search procedure across fixed-budget AI R&D tasks; its ignition test was inconclusive, so the paper does not establish that the improved agent recursively outperforms its improver. The cache-engineering paper reports a held-out recursively improved cache policy. None of these results alone establishes general RSI or autonomous improvement without external feedback.

## Questions for evaluating a self-improvement claim

- **What changes?** Separate model weights from prompts, tools, memory, harness code, and runtime policy.
- **What experience drives revision?** Identify whether the signal is task trajectory, test feedback, reference answers, or an external evaluator.
- **When does a revision take effect?** Distinguish offline training, between-task updates, between-attempt revisions, and hot reload during a live task.
- **What state and budget persist?** A new policy can appear better if it receives more time, resets context, retains useful artifacts, or is evaluated on different data.
- **How is improvement verified?** Look for held-out tasks, correctness gates, repeated runs, and protection against reward hacking or measurement leakage.
- **What is the transfer scope?** Within-task adaptation is not the same claim as cross-task generalization; a benchmark result is not general intelligence or autonomous research capability.

## Systems trade-offs and safeguards

Online revision can use task-specific evidence quickly, but may introduce faulty or insecure behavior. Kernel-managed validation and rollback, as in [[BOXBOX]], address runtime continuity; that paper does not claim they form a complete security boundary. Continual Harness demonstrates a distinct coupling between an online-refined harness and model training, but the tested student relies on a frontier teacher and results vary with model capability. AIDE2's offline candidate grading avoids changing the live task runtime, but its nested evaluation noise can select a worse incumbent. Evaluation should account for the quality of the revised procedure and the cost, correctness, safety, capability requirements, and continuity of the full execution process. See [[Recursive Self-Improvement Mechanisms]] for comparison across update targets and timescales.

For harness revision strategies, see [[Online and Offline Harness Improvement]]. For the separate question of agent workflow metadata interacting with inference-engine state, see [[Harness–Engine Coordination]].
