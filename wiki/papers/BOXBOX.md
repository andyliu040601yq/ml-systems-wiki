# BOXBOX: Recursive Harness Self-Improvement for Long-Running Performance Optimization

**Source:** [[raw/papers/2611.09218v1.pdf|BOXBOX paper]] (Tan et al., arXiv:2611.09218v1, 2026)

**Related:** [[Agent Self-Improvement]], [[Online and Offline Harness Improvement]], [[Agentic Inference Optimization Evaluation]], [[Harness–Engine Coordination]]

## Problem and contribution

An agent optimizing a real software project may encounter task-specific obstacles that its original tools and workflow cannot handle. Offline harness revisions may not transfer to a new project, while an external controller that revises the harness between attempts cannot react during the current execution. BOXBOX studies whether the task-solving agent can decide when to improve its own harness using evidence from its ongoing trajectory.

The paper contributes two pieces:

- **PERFRACE**, a benchmark of 100 long-running, open-ended software performance tasks over real projects and fixed workloads. Tasks cover seven domains, including developer tools, data systems, scientific computing, machine learning, and security. Agents repeatedly submit project snapshots to a separate judge; incorrect submissions score zero, and the objective is best-so-far correctness-preserving speedup within a four-hour budget.
- **BOXBOX**, a runtime that lets the agent request a harness review while its task is in progress. The review agent can modify the full executable harness (guidance, tools, control flow, or state management), rather than only a predefined component.

## Runtime and improvement loop

The agent invokes `boxbox()` when it judges that improving its harness may be worth the remaining time. The kernel suspends the task and preserves the trajectory, terminal session, and workspace. A review session diagnoses repeated work, failures, missing capabilities, or useful patterns, then commits a candidate revision. The kernel validates and hot-reloads a valid revision, reattaches the prior execution context, and resumes the same task. If validation or activation fails, the previous runtime remains active. Runtime faults can also trigger recovery review.

This is **online, task-specific harness improvement**. Harness changes affect subsequent agent actions and tool use; the agent's model weights are not updated. It is therefore a concrete RSI mechanism at the level of an agent's executable procedure, not evidence of general autonomous scientific discovery or recursive model training. See [[Agent Self-Improvement]] for the distinction.

## Evaluation and results

On 50 held-out PERFRACE tasks, with three runs per method and a shared four-hour task budget, the paper reports:

| Method | Speedup@4h | Mean model cost/run | Speedup/$ |
| --- | ---: | ---: | ---: |
| Frozen harness | 1.442× | $23.22 | 0.2142 |
| Offline Harness Evolution | 1.281× | $24.56 | 0.1243 |
| Harness Scaling (external revisions between attempts) | 1.539× | $12.98 | 0.2872 |
| BOXBOX | **1.702×** | $16.43 | **0.4886** |

The aggregate uses the harmonic mean of each task's geometric mean across runs, so these numbers describe this benchmark and protocol rather than a general agent capability ranking. BOXBOX's mean cost is higher than Harness Scaling's, but the paper reports higher speedup per dollar under its task-level metric. A 14-task ablation also favors agent-triggered reviews and full-harness editing over scheduled reviews and restricted editing; it is smaller than the main evaluation.

BOXBOX makes a median of five reviews per run. Reviews consume a median 2.1% of task time and 13.8% of model cost. The revision categories include prompt rules, skills, tools, execution logic, and fault fixes. These are changes to how the agent works, not direct project-code improvements; task performance still comes from the agent's project edits and evaluations.

## Systems and evaluation lessons

- **Keep the task state across revisions.** BOXBOX preserves trajectory, workspace, and terminal state across runtime replacement. The paper contrasts this with Harness Scaling's new agent sessions, which retain project-code progress but reset execution context.
- **Put the trigger where the evidence is.** The task-solving agent sees emerging difficulties and allocates its shared time budget between project work and harness review. The paper's ablation suggests this can outperform fixed periodic reviews in its setup.
- **Make activation a runtime responsibility.** The separate kernel validates revisions, retains a working runtime on failure, and manages hot reload. This reduces disruption from faulty edits but is not a security proof.
- **Long-running optimization needs process metrics.** PERFRACE tracks best-so-far correct speedup over time, rather than only an end-state task-success result. The hidden judge and shadow verification are intended to protect correctness and scoring integrity.

## Limitations and safety boundary

PERFRACE is a curated benchmark (100 tasks), and the headline comparison uses 50 held-out tasks with three runs each. The reported evidence supports BOXBOX under those workloads, model, harness, budget, and evaluation choices; it does not establish transfer to other agent domains or RSI in general. Offline harness evolution performs poorly here, but that does not show offline approaches are universally ineffective.

Because the agent edits executable harness code and operates on real repositories, revisions can weaken safeguards or introduce unsafe behavior. The authors use isolated research environments, validation, rollback, and separate shadow evaluation, while explicitly stating these are not a complete security boundary. They characterize BOXBOX as a research prototype, not a production-ready self-modifying framework.

## Connections

[[Cache Engineering for Agents]] studies a different improvement target: a held-out policy for changing which KV states participate in future attention. BOXBOX changes the agent's executable harness during one ongoing software task. Both are examples of recursive improvement in an agent system, but neither updates model weights, and their metrics and mechanisms are not directly comparable. [[Online and Offline Harness Improvement]] places BOXBOX alongside fixed/offline and externally scheduled harness revision. [[Harness–Engine Coordination]] concerns a separate boundary: exchanging workflow intent and serving state, rather than revising the task harness itself.
