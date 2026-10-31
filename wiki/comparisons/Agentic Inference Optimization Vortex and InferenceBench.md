# Agentic Inference Optimization: Vortex and InferenceBench

**Sources:** [[Vortex]], [[InferenceBench]], [[HEAR Protocol]]

**Related:** [[Agentic Inference Optimization Evaluation]]

These works expose different system boundaries and evaluate different claims. Vortex and InferenceBench study agents optimizing systems; HEAR studies a reusable harness–engine protocol through two policy instantiations, rather than an autonomous optimization benchmark.

| | Vortex agent experiments | InferenceBench | HEAR |
| --- | --- | --- |
| Search/decision scope | Generate/iterate sparse-attention flows inside a documented DSL and serving stack | Choose and deploy an end-to-end server, framework, backend, precision, and runtime settings | Coordinate queue/cache scheduling and role-specific engine-mode selection from harness and engine state |
| Main evidence | Agents can produce diverse useful flows; an 18-hour loop reaches 3.46x throughput at similar AIME24 accuracy in the reported Qwen3 setup | Under two hours, agents beat default baselines but matched-budget search wins the aggregate; valid final deployment and integrity gates matter | Guarded cache-aware scheduling reaches 1.61x batch speedup on SCBench; workload-specific modes reach 1.23x and 2.45x on two agent benchmarks |
| Failure surface | Candidate algorithm quality and end-to-end accuracy/throughput trade-off | Crashes, failed quality/integrity checks, narrow exploration, measurement exploits, and loss of a good intermediate configuration | Stale observations, unsupported controls, overload-sensitive retention, and workload-specific mode trade-offs |
| What it establishes | A suitable abstraction can make a domain-specific algorithm space easier for agents to explore | Domain knowledge alone does not guarantee disciplined broad search or a valid best final server | A shared interaction contract can support multiple execution policies; results depend on those policies and engine support |

## Synthesis

Vortex reduces systems-engineering friction by constraining the task to a composable, documented sparse-attention interface. InferenceBench intentionally leaves the action space broad and reveals the cost of coordinating framework choices, experiments, measurement, and final deployment. HEAR addresses a different gap: the runtime interface between the harness and engine, where workflow dependencies/context lifecycles and live cache/queue/capability state must meet. Its contract could support policies used by either kind of system, but the paper does not show it as a benchmark or integrate it with Vortex/InferenceBench.

Neither result alone demonstrates general autonomous systems research. Vortex's findings are tied to sparse-attention workflows and its tested models; InferenceBench's results are tied to single-H100, single-node tasks, chosen agents/scaffolds, and the benchmark's gates; HEAR's speedups are tied to two policy instantiations and the paper's evaluated setups. The numbers should not be ranked across these different questions.
