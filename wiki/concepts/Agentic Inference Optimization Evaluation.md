# Agentic Inference Optimization Evaluation

**Sources:** [[InferenceBench]], [[Vortex]]

Evaluating an agent that optimizes inference systems requires more than measuring its best observed latency. The task also includes deployability, model quality, valid measurement, search behavior, and preservation of the best configuration.

## Evaluation dimensions

| Dimension | What to measure | Example from the sources |
| --- | --- | --- |
| Optimization target | Which bottleneck and workload is scored? | [[InferenceBench]] separates prefill TTFT, decode TPOT, concurrent throughput, and a balanced objective. |
| Action space | Is the agent optimizing one exposed algorithm or composing a full system? | [[Vortex]] provides vFlow/documentation for sparse-attention design; [[InferenceBench]] allows framework, backend, quantization, and runtime choices. |
| Quality and correctness | Does the optimized system preserve useful model behavior? | InferenceBench gates on at least 95% of baseline MMLU-Pro accuracy; Vortex tracks benchmark quality/accuracy frontiers. |
| Measurement integrity | Does the score correspond to actual inference? | InferenceBench audits final server state and run traces for evaluation-path manipulation. |
| Search process | Does the agent propose distinct hypotheses, isolate variables, measure, and compare? | InferenceBench logs traces/configurations and finds shallow, uncontrolled search despite domain knowledge. |
| Shipped state | Is the best measured configuration still runnable at the deadline? | InferenceBench scores failed final deployments at baseline utility and separately analyzes best-seen intermediate servers. |

## Reading results carefully

Agent studies with different interfaces answer different questions. Vortex's agent experiments test whether an expressive, constrained DSL makes sparse-attention algorithm exploration productive. InferenceBench tests open-ended autonomous inference engineering under a time budget, where deploying and preserving a valid server are part of the task. Compare these as complementary evaluation regimes, not as a head-to-head score.

Report workload, hardware, baseline, time budget, failure penalties, seed variance, quality threshold, and integrity checks with any headline speedup. A single aggregate can conceal different behavior across prefill, decode, and queueing; scenario-level outcomes are needed to understand what changed.

## Design lessons

- Use independent development and evaluation inputs where possible to reduce optimization against the callable evaluator.
- Score valid final artifacts and preserve a trace of intermediate runs so search capability can be separated from shipping reliability.
- Include integrity monitoring when the reward is an unbounded performance metric.
- Treat equal wall-clock budgets as a comparison of complete workflows. A search method may outperform an agent even if the agent can occasionally find a stronger intermediate configuration.
- Keep scope explicit: single-GPU server tuning does not establish cluster-scale or multi-GPU optimization capability.
