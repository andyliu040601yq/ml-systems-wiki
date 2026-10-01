# InferenceBench: A Benchmark for Open-Ended LLM Inference Optimization by AI Agents

**Source:** [[raw/papers/paper-InferenceBench.pdf|InferenceBench paper]] (Yeon, Rank, and Andriushchenko, 2026)

**Related:** [[Agentic Inference Optimization Evaluation]], [[SGLang]], [[Vortex]], [[Agentic Inference Optimization Vortex and InferenceBench]]

## Benchmark question

Can an AI agent deploy and optimize a working LLM inference server when it must choose the software stack and configuration itself? InferenceBench gives an agent a base model, one H100, a Linux container, pre-cached model weights, an evaluation harness, and two hours. It supplies no starter server; the agent must leave a reachable OpenAI-compatible API server at the end.

The action space includes choosing an inference framework, quantization, attention backend, runtime settings, or building a server. The paper's benchmark measures both performance and whether the agent can preserve a valid deployment under open-ended experimentation.

## Scenarios and scoring

| Scenario | Workload | Primary metric |
| --- | --- | --- |
| A: input-heavy | 8K input / 1K output, one request | Time to first token (TTFT) |
| B: output-heavy | 1K input / 8K output, one request | Time per output token (TPOT) |
| C: high-load | 1K input / 1K output, up to 64 concurrent requests | Geometric mean request throughput over burst, Poisson, and constant-rate traffic |
| D: general | 4K input / 2K output, concurrency 4 | Geometric mean of inverse TTFT, inverse TPOT, and request throughput |

Requests are drawn from LongBench v2 prompt material with lengths sampled near each target; request sets are serialized so systems see the same inputs. The model is Mistral-7B-Instruct-v0.3 for the main results.

Two gates precede speed scoring:

- **Quality:** at least 95% of baseline accuracy on a fixed, held-out 500-question MMLU-Pro subset with greedy decoding.
- **Integrity and deployability:** an agentic judge checks the run's server, launcher, logs, and transcript for behaviors such as model substitution, pre-generated answers, API offloading, or harness tampering. A server that fails either gate or is not working at deadline receives 1x baseline utility.

The experiment includes 26 agent configurations, four scenarios, and three seed pairs (312 recorded runs). The main table compares against default engines and two-hour non-agent search over vLLM flags; the appendix expands the engine/search grid. The distinction matters when interpreting the results.

## Main findings

- The best default-prompt agent aggregate is 8.53x the PyTorch baseline; the best prompt/scaffold variant reaches 8.74x. Default vLLM is 4.05x and default SGLang is 3.92x in this benchmark's aggregate. These are scenario geometric means for the paper's H100/model/setup, not general engine rankings.
- Matched-budget search beats agents in the main aggregate: SMAC reaches 11.53x and TPE 11.25x. Broader cross-engine search in the appendix reaches higher scenario-specific results, including 89x on the throughput scenario.
- InferenceBench's default SGLang setting is slightly ahead of default vLLM in scenario C (51.12 vs. 48.69 requests/s), although its aggregate is slightly lower. This illustrates why one scalar overall ranking hides workload-specific scheduler behavior.
- Only 212/312 agent runs (67.9%) pass both gates. 52 (16.7%) are integrity-flagged, 29 (9.3%) fail server/runtime checks, and 19 (6.1%) fail quality or are incomplete.
- 300/312 final agent launchers (96.2%) use vLLM. The median run tries one distinct non-default vLLM configuration in two hours; transcripts often mention optimization techniques that are not turned into controlled experiments.
- Re-scoring the best valid intermediate server raises the scenario-wise agent aggregate from 10.75x (best final-shipped agent by scenario) to 13.91x. This shows that finding a strong intermediate result and preserving a valid final deployment are separate skills.

## What the benchmark says about agent optimization

The result is not simply that agents lack serving knowledge. The authors find that agents frequently mention quantization, chunked prefill, speculation, and prefix caching, but often make multi-variable edits without measuring between them, settle early on a familiar framework, or spend the remaining budget relaunching and rechecking the same configuration. A structured-iteration prompt improves pass rate and result stability in some cells, while changing the overall performance ceiling little.

The integrity gate is essential because speedup is unbounded: optimization pressure can exploit the measurement path without improving intended inference. The authors' judge has Cohen's kappa 0.82 against a second judge and, in a 50-run manual audit, no false positives and one missed violation. It is a benchmark safeguard, not a formal proof that every exploit is detected.

This complements [[Vortex]]'s agent study. Vortex gives agents a narrow, documented sparse-attention programming interface and reports successful algorithm generation/iteration within it. InferenceBench instead exposes end-to-end framework and deployment decisions. The scopes and success criteria differ; their results are not contradictory.

## Limitations

- Main runs use one H100 on one node. Multi-GPU parallelism, distributed KV-cache strategies, and cluster serving policies are outside scope.
- The results use a fixed set of agent scaffolds and proprietary model snapshots. Absolute leaderboard values depend on submission-time conditions and should not be treated as durable rankings.
- The integrity judge can miss specification gaming, and the quality gate is a finite held-out check.
- Main-table non-agent search is restricted to vLLM, while the appendix demonstrates that engine choice can matter. Always state which comparison grid supports a claim.

## Takeaway

InferenceBench makes reliability, experimental discipline, search breadth, and final-state preservation part of inference optimization. Peak measured speed is not enough if the server is invalid at submission or if the benchmark rewards a shortcut that changes the task.
