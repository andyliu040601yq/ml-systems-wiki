# Agentic Inference Optimization: Vortex and InferenceBench

**Sources:** [[Vortex]], [[InferenceBench]]

**Related:** [[Agentic Inference Optimization Evaluation]]

These papers study AI agents in inference-systems work, but they give the agent different interfaces and score different parts of the process.

| | Vortex agent experiments | InferenceBench |
| --- | --- | --- |
| Search scope | Generate/iterate sparse-attention flows inside a documented DSL and serving stack | Choose and deploy an end-to-end server, framework, backend, precision, and runtime settings |
| Main evidence | Agents can produce diverse useful flows; an 18-hour loop reaches 3.46x throughput at similar AIME24 accuracy in the reported Qwen3 setup | Under two hours, agents beat default baselines but matched-budget search wins the aggregate; valid final deployment and integrity gates matter |
| Failure surface | Candidate algorithm quality and end-to-end accuracy/throughput trade-off | Crashes, failed quality/integrity checks, narrow exploration, measurement exploits, and loss of a good intermediate configuration |
| What it establishes | A suitable abstraction can make a domain-specific algorithm space easier for agents to explore | Domain knowledge alone does not guarantee disciplined broad search or a valid best final server |

## Synthesis

Vortex reduces systems-engineering friction by constraining the task to a composable, documented sparse-attention interface. InferenceBench intentionally leaves the action space broad and reveals the cost of coordinating framework choices, experiments, measurement, and final deployment. The findings therefore complement each other: a programmable substrate may help agents explore an algorithm family, while whole-server autonomy still requires controlled experiments, integrity safeguards, and reliable state management.

Neither result alone demonstrates general autonomous systems research. Vortex's findings are tied to sparse-attention workflows and its tested models; InferenceBench's results are tied to single-H100, single-node tasks, chosen agents/scaffolds, and the benchmark's gates.
