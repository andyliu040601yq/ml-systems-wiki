# Agentic Inference Optimization Evaluation

**Sources:** [[ServerBench]], [[InferenceBench]], [[Vortex]], [[BOXBOX]], [[HEAR Protocol]], [[TAO]]

Evaluating an agent that optimizes inference systems requires more than measuring its best observed latency. The task also includes deployability, model quality, valid measurement, search behavior, and preservation of the best configuration.

## Evaluation dimensions

| Dimension | What to measure | Example from the sources |
| --- | --- | --- |
| Optimization target | Which bottleneck and workload is scored? | [[InferenceBench]] separates prefill TTFT, decode TPOT, concurrent throughput, and a balanced objective. |
| Workload structure and operating point | Does the workload retain dependencies and does the agent choose the load under service constraints? | [[ServerBench]] replays shared-context, response-dependent coding sessions and lets the agent select session concurrency against TTFT/TPOT targets. |
| Service value | Does the objective include which work meets latency targets and the cost of serving it? | ServerBench scores latency-qualified token value per GPU dollar, rather than unconstrained throughput or speedup alone. |
| Action space | Is the agent optimizing one exposed algorithm or composing a full system? | [[Vortex]] provides vFlow/documentation for sparse-attention design; [[InferenceBench]] allows framework, backend, quantization, and runtime choices. |
| Quality and correctness | Does the optimized system preserve useful model behavior? | InferenceBench gates on at least 95% of baseline MMLU-Pro accuracy; Vortex tracks benchmark quality/accuracy frontiers. |
| Measurement integrity | Does the score correspond to actual inference? | InferenceBench audits final server state and run traces for evaluation-path manipulation. |
| Search process | Does the agent propose distinct hypotheses, isolate variables, measure, and compare? | InferenceBench logs traces/configurations and finds shallow, uncontrolled search despite domain knowledge. |
| Shipped state | Is the best measured configuration still runnable at the deadline? | InferenceBench scores failed final deployments at baseline utility and separately analyzes best-seen intermediate servers. |
| Long-horizon progress | Does the agent make correctness-preserving progress throughout a long task, and can it revise its own workflow when blocked? | [[BOXBOX]] tracks best-so-far correct speedup on PERFRACE and allows agent-triggered harness changes during execution. |
| Full-system verification | Does the evaluation exercise the submitted system's real execution paths on held-out workloads? | ServerBench rebuilds in a fresh deployment and checks prompt/output completeness, streaming, checkpoint fidelity, held-out answers, and post-run behavior. |

## Reading results carefully

Agentic systems papers can study either the agent as an optimizer or the serving system used by agents. Vortex's agent experiments test whether an expressive, constrained DSL makes sparse-attention algorithm exploration productive. InferenceBench tests end-to-end inference engineering under preset scenarios; ServerBench tests full server design under stateful coding traces, latency targets, and an economic service objective. BOXBOX studies long-running software optimization and online improvement of the agent's executable harness. HEAR evaluates two particular harness–engine coordination policies; TAO evaluates a tool-aware multi-node scheduler. The latter two are serving-design studies, not open-ended agent-optimization benchmarks. Compare these as complementary questions, not as a head-to-head score.

Report workload, hardware, baseline, time budget, failure penalties, seed variance, quality threshold, and integrity checks with any headline speedup. A single aggregate can conceal different behavior across prefill, decode, and queueing; scenario-level outcomes are needed to understand what changed.

## Design lessons

- Use independent development and evaluation inputs where possible to reduce optimization against the callable evaluator.
- Score valid final artifacts and preserve a trace of intermediate runs so search capability can be separated from shipping reliability.
- Include integrity monitoring when the reward is an unbounded performance metric.
- Treat equal wall-clock budgets as a comparison of complete workflows. A search method may outperform an agent even if the agent can occasionally find a stronger intermediate configuration.
- Keep scope explicit: InferenceBench uses one H100, while ServerBench covers three specified deployments; neither alone establishes general optimization across clusters, models, or arrival patterns.
- Separate a coordination interface from the policies evaluated through it: HEAR's results support its tested policies/workloads, not a claim that message semantics alone improve serving.
