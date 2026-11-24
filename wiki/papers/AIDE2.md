# Recursive Self-Improvement of AI Research Agents: AIDE2

**Source:** [[raw/papers/2609.26457v1.pdf|AIDE2 paper]] (Srikanth et al., arXiv:2609.26457v1, 2026)

**Related:** [[Agent Self-Improvement]], [[Recursive Self-Improvement Mechanisms]], [[BOXBOX]], [[Continual Harness]], [[Online and Offline Harness Improvement]]

## Research question

Can an AI research agent improve the research procedure that drives its own future optimization? AIDE2 targets the **agent harness/code**, not model weights. Its central idea is a bi-level loop: an inner agent optimizes code for AI R&D tasks, while an outer agent rewrites the inner agent and retains rewrites that improve its task performance under a fixed evaluation budget.

## Two-level optimization loop

### Inner loop: solve research-engineering tasks

An inner agent receives a codebase and a measurable objective, proposes and executes candidate edits, and returns one solution within a fixed dollar budget that counts both model tokens and solution execution. In the selection benchmark it works across:

- ML engineering;
- heuristic algorithm engineering; and
- harness engineering (prompts, context management, and feedback loops).

The inner agent uses task-public feedback while it searches. Its returned solutions are graded on private held-out task data, and the aggregate private score forms the agent grade $g(a)$. Thus the signal used to guide search is separated from the signal used to select agent rewrites.

### Outer loop: improve the optimizer

The outer-loop agent edits the incumbent research-agent code. Each proposed candidate runs through the same selection benchmark and per-task budgets. The loop keeps the best graded agent as incumbent, so an accepted rewrite becomes the code that future outer-loop steps modify.

In the reported implementation, AIDE0 is a tree-search optimizer with `draft`, `debug`, and `improve` operators and greedy selection of the highest-scoring candidate. AIDEhuman, a production agent developed through human-driven R&D, drives the outer loop; Claude Opus 4.7 is used there, while Gemini 3 Flash evaluates the inner agents. The model is held fixed within each loop: the recursive updates are agent-code/harness rewrites, not weight training.

## Eight-day run and discovered changes

The paper reports one autonomous 8-day trajectory with 100 outer-loop candidate nodes and seven accepted rewrites. Under a fixed selection-benchmark evaluation budget, the incumbent private grade rises from 0.703 to 0.778; the AIDEhuman comparison grade is 0.749. The accepted changes address search plateaus, context growth, and robustness.

The final AIDE85 agent replaces greedy search with a UCB1 bandit over five drafting strategies, with periodic forking of the global best under a different strategy to escape stagnation. It bounds prompt growth by using compact summaries instead of full candidate history and adds failure memory when the candidate bug rate reaches a threshold. It also adds prompt-level robustness instructions and repairs a verified evaluator bug rather than exploiting it. On held-out tasks, the paper reports median per-call prompt-size reductions of about 7× on MLE-Bench, over 40× on WeatherBench 2, and about 50× on ALE-Bench and FML-Bench.

## Transfer and behavioral evidence

The authors evaluate AIDE0, intermediate AIDE47, final incumbent AIDE85, and AIDEhuman on four external benchmarks never used to select the rewrites:

| Benchmark | Task-family relation to selection | AIDE85 | AIDEhuman | Reported interpretation |
| --- | --- | ---: | ---: | --- |
| ALE-Bench | In-family, held-out tasks | 1790 | 1511 | AIDE85 higher contest score |
| MLE-Bench | In-family, held-out tasks | 0.722 | 0.708 | AIDE85 higher private percentile |
| FML-Bench | In-family, held-out tasks | 19.9% | 19.6% | Similar mean test improvement |
| WeatherBench 2 | Out-of-distribution physics-based forecasting | 0.793 | 0.404 | AIDE85 higher forecast-skill gain |

These are the paper's reported benchmark means; the three in-family benchmarks use task instances that do not overlap with selection. WeatherBench 2 is the OOD task family. The outcomes support transfer of the discovered search procedure, but do not establish transfer to arbitrary research domains.

On a separate held-out kernel-engineering evaluation, reward hacking declines from 55% for AIDE0 to 32% for AIDE85; AIDEhuman is at 39%. This behavior was not directly optimized by the selection grade. The result is an observed lineage-level change on 38 held-out kernel/training-context pairs; the paper does not identify which accepted rewrite caused it.

## What the result does and does not establish

The paper's **ignition test** asks whether the improved agent can drive a subsequent RSI loop better than AIDEhuman. Two arms, each with three 50-step seeds and the same AIDE47 starting point, end at similar mean grades (0.780 for AIDE47 and 0.782 for AIDEhuman). The authors call the evidence inconclusive; it does not demonstrate accelerating or self-sustaining improvement.

Evaluation noise compounds across inner-task search and outer-agent grading. A falsely accepted rewrite can change the incumbent and distort later search. The paper reports a single main 8-day trajectory, and the stronger ignition/generalization claims would require expensive additional outer-loop seeds. The resulting agent is also complex and difficult to interpret or deploy in systems with compatibility constraints.

## Connections

[[BOXBOX]] lets an agent revise its executable harness in response to an ongoing software-optimization trajectory, but does not evaluate model-weight training or recursive improvement of the outer optimizer. [[Continual Harness]] adapts harness state during an embodied task and additionally trains model weights between iterations using a frontier teacher. AIDE2 instead runs a cross-task outer loop that rewrites the research optimizer itself and selects it on private held-out task grades. These are distinct levels of the [[Recursive Self-Improvement Mechanisms]] taxonomy.
