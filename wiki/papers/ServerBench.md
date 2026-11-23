# ServerBench: How Can Coding Agents Design Specialized LLM Serving Systems?

**Source:** [[raw/papers/2611.06841v1.pdf|ServerBench paper]] (Zhong et al., arXiv:2611.06841v1, 2026)

**Related:** [[Agentic Inference Optimization Evaluation]], [[InferenceBench]], [[Vortex]], [[ServerBench and Agentic Inference Optimization Benchmarks]]

## Research question

Can coding agents design a complete LLM serving system for a specified model, GPU allocation, coding-agent workload, and latency service requirements? ServerBench treats both the server implementation and offered session concurrency as design decisions. The agent can change batching, scheduling, execution, memory management, and state reuse while preserving the checkpoint and inference semantics.

The motivation is that isolated requests and fixed batch tests underrepresent coding-agent serving. Real traces contain dependent turns, shared context, parallel subagent requests, mixed prompt lengths, and recorded delays. These properties make latency, prefix reuse, memory pressure, and prefill/decode interference interact with how much work each session can complete.

## Task, workload, and objective

Each task starts from a minimal Transformers server and fixes the model checkpoint, GPU allocation, reconstructed coding-session workload, token prices, and TTFT/TPOT targets. The agent gets an eight-hour development budget and submits a server implementation plus its chosen session concurrency.

The workload replays AgentX coding sessions. Privacy-preserving reconstruction replaces prompt and tool contents while retaining request lengths, shared prefixes, session dependencies, parallel branches, and inter-turn delays. Response-dependent turns wait for their prerequisites; global idle intervals are compressed while preserving event order and dependencies. Development and final replay use disjoint seeds from the same underlying corpus.

The objective, **dollars per dollar**, is the token value of completed requests that satisfy both per-request TTFT and TPOT thresholds, divided by allocated GPU cost during the measurement window. Token values use fixed input/output API prices. The latency thresholds are calibrated from model-specific SGLang reference configurations before development, using the maximum observed seed-level p90 targets. Thus the score jointly rewards serving more valuable work, meeting latency requirements, and using GPU time efficiently. It measures deployment efficiency under fixed token prices; it is not realized profit and does not include the coding agent's development/API cost.

## Evaluation and verification

The study tests 11 coding-agent models across three deployments: Qwen3.5 9B on 2×L40, Gemma E4B on 1×L40, and Qwen3.8 27B INT4 on 2×L40. Each agent–deployment pair receives one eight-hour development run. Final evaluation rebuilds the submitted implementation in a fresh deployment and uses held-out replay seeds.

Only verified systems score. Checks cover full prompt delivery, prescribed output lengths, streaming consistency, session progress, generation fidelity to the checkpoint, and held-out answer accuracy. Separate EOS probes test natural stopping; code/action review checks for prohibited shortcuts and inference-semantic changes. This matters because custom scheduling, speculative decoding, and state reuse can create execution paths that short or isolated tests miss.

## Findings

- Across 33 agent–deployment runs, 22 produce verified systems and 13 exceed the strongest evaluated baseline.
- Claude Sonnet 5.5 reaches 2.97 dollars per dollar on Gemma E4B, 4.87× the vLLM reference score in that deployment. The paper reports results across all three model/hardware configurations; these are not a universal ranking of agents or engines.
- Stronger designs coordinate server capacity and offered load. Agents first improve capacity, then increase concurrency; when extra sessions cause TPOT violations, some agents back off to a lower load.
- Workload-relevant tests guide better decisions. In one prompt-lookup speculation example, a random-text probe failed to reveal value, while experiments using replay replies showed benefit and led the agent to retain the optimization.
- Verification must exercise deployed workload paths. Short or isolated checks missed long-context stalls and cross-request crashes; some known faults persisted or returned after edits.

## Limitations and interpretation

The evaluation covers three model/GPU deployments and one coding-trace source. Development and final evaluation use different seeds from the same reconstructed corpus; this limits seed leakage but does not establish transfer to different trace populations. Reconstructed prompts preserve workload structure but are not the original coding tasks. Closed-loop replay also differs from open-loop production arrivals.

The reported comparisons characterize the submitted artifacts and selected trajectories; they do not establish causal success rates for particular design choices. Workload variability and agent-run variability require repeated runs to separate. The fixed token-price objective excludes agent development costs and other operating expenses. Behavioral tests and code review have finite coverage, and SGLang-derived latency calibration influences the operating target.

## Connections

[[InferenceBench]] also asks agents to optimize complete inference servers, but uses preset scenarios and a speedup/quality/integrity evaluation. ServerBench instead replays stateful, response-dependent coding sessions and lets the agent choose concurrency under latency targets, scoring latency-qualified token value per GPU dollar. [[Vortex]] gives agents a more constrained, programmable sparse-attention design space. These benchmarks probe different scopes and objectives; see [[ServerBench and Agentic Inference Optimization Benchmarks]].

ServerBench's capacity/load co-design connects to [[Tool-Aware Joint Scheduling]] and [[Harness–Engine Coordination]], though ServerBench evaluates agent-generated serving systems rather than implementing TAO or HEAR. Its emphasis on independent verification also complements [[Agentic Inference Optimization Evaluation]].
