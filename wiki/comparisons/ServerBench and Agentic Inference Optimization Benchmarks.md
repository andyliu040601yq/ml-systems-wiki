# ServerBench and Agentic Inference Optimization Benchmarks

**Papers:** [[ServerBench]], [[InferenceBench]], [[Vortex]]

These benchmarks study agents improving inference systems at different system boundaries. Their scores answer different questions and should not be compared as a shared leaderboard.

| Dimension | Vortex agent experiments | InferenceBench | ServerBench |
| --- | --- | --- | --- |
| Agent's design space | Generate and iterate sparse-attention algorithms inside a documented vFlow DSL and serving substrate | Choose/deploy an inference stack and configuration, including framework, backend, precision, and runtime settings | Design a complete specialized server and choose application-session concurrency for a fixed model, GPU allocation, and service workload |
| Workload | Sparse-attention generation workloads, including long decode and quality evaluation | Four preset input-heavy, output-heavy, high-load, and general inference scenarios on one H100 | Stateful coding-agent traces with shared prefixes, dependent turns, parallel subagents, and recorded delays, on three model/GPU deployments |
| Objective and gates | Throughput/latency and model-quality trade-offs for the sparse-attention task | Scenario performance with model-quality, integrity, and final-server deployability checks | Token value from completed requests meeting both TTFT and TPOT targets, divided by GPU cost; independent fidelity, correctness, workload, and review checks gate scoring |
| Agent budget/setup | Reported 18-hour agent loop in the highlighted Qwen3 setup | Two-hour autonomous deployment task | Eight-hour server development run followed by a fresh deployment and held-out-seed replay |
| Representative evidence | An 18-hour loop reaches 3.46× dense-attention throughput at similar AIME24 accuracy in the paper's Qwen3 setup | Agents beat default baselines in some settings, but matched-budget search wins the main aggregate; only 67.9% of recorded runs pass both gates | 22 of 33 agent–deployment runs produce verified systems; 13 exceed the strongest baseline. Sonnet 5.5 reaches 4.87× the vLLM reference's dollars-per-dollar score on Gemma E4B |
| Main lesson | A suitable programming abstraction makes domain-specific algorithm exploration tractable | Domain knowledge alone does not ensure disciplined broad search or a valid final server | Stateful workload structure, concurrency choice, latency-qualified service value, and verification of real execution paths all shape success |

## Synthesis

Vortex narrows the search space to a programmable sparse-attention substrate. InferenceBench broadens the agent's choices across inference frameworks and server configuration but evaluates preset workload scenarios. ServerBench also permits whole-server design, and adds response-dependent coding traces, agent-selected concurrency, and a service objective that includes latency qualification and GPU cost. Its reconstructed coding workload retains trace structure but not original prompt/tool contents; development and final draws are seed-disjoint samples of the same corpus.

The evaluation designs emphasize different failure modes. Vortex's central concern is the quality/performance trade-off of generated attention algorithms. InferenceBench emphasizes model quality, measurement integrity, and whether an optimized server remains runnable at the deadline. ServerBench additionally checks complete prompt delivery, output/streaming behavior, checkpoint fidelity, held-out answers, and workload execution in a fresh deployment. These checks remain finite, and ServerBench's fixed prices and GPU-cost objective do not include coding-agent development expense.

For detailed dimensions such as integrity, workload coverage, and search process, see [[Agentic Inference Optimization Evaluation]]. [[HEAR Protocol]] and [[TAO]] concern serving interfaces and scheduling policies rather than agentic system-design benchmarks; their systems may provide mechanisms an agent could study, but none of these benchmark papers evaluates that composition.
