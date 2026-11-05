# TAO: Tool-Aware Joint Scheduling for LLM-Based Agentic Serving

**Source:** [[raw/papers/2610.37230v1.pdf|TAO paper]] (Li et al., arXiv:2610.37230v1, 2026)

**Related:** [[Tool-Aware Joint Scheduling]], [[Harness–Engine Coordination]], [[Paged KV Cache]], [[SGLang]], [[HEAR Protocol]], [[Agentic Serving Scheduling HEAR and TAO]]

## Problem: intermittent GPU demand, persistent sessions

An agent session alternates model turns with external tool calls. In the paper's Clustered-GPUs with Distributed-CPUs (CGDC) setting, tools execute on distributed CPUs while clustered GPU nodes serve inference. During a tool call, the session does not need inference, but may need GPU capacity again as soon as the tool returns. Keeping it resident can waste capacity; releasing it can lose node-local prefix KV and make the next turn expensive to prefill.

Existing schedulers often score sessions individually and then route them with heuristics. That can miss near-term tool returns and make locally reasonable placements that consume capacity needed by a more valuable combination of sessions. TAO separates **session evaluation** from **joint admission and placement**.

## Evaluation signal and scheduling algorithm

At a restart decision, TAO considers waiting sessions (new or interrupted), eligible GPU nodes, each session's token-based admission footprint, and residual node capacities. For session $i$ on node $b$, it constructs a lexicographic value:

\[
v_{i,b}=(k_{i,b},r_i,t_i)
\]

- $k_{i,b}$: node-specific reusable-prefix KV credit. Observed cache credit decays with age as $k_{i,b}(t)=k^0_{i,b}2^{-\tau/h}$; the evaluated half-life is 10 seconds. The scheduler values opportunities but does not manage physical KV itself.
- $r_i$: binary near-term tool-return readiness. Observed tool completion or tool progress/history predicts whether a session is likely to need inference within the next 1-second horizon. Direct timing evidence is preferred; history-based estimates and a fallback are used when direct deadlines are absent.
- $t_i$: accumulated token footprint, used only after the higher-priority coordinates tie and considered alongside the capacity cost.

TAO lexicographically maximizes the aggregate triplets under per-node capacity and eligibility constraints: fresh cache opportunity takes precedence, then readiness, then token footprint. This explicit precedence avoids a fixed weighted sum whose weights would need hand-tuning across signals.

The decision stage is a capacity-state dynamic program for a multiple-knapsack/generalized-assignment problem. It jointly chooses which sessions to admit and where to place them, accounting for competition among assignments. Capacity is discretized, so the DP is exact for the frozen discretized snapshot, not necessarily for the original continuous-capacity problem. It runs asynchronously; before application, the executor revalidates the plan against live state. When a state budget is exhausted, the implementation falls back to best-fit-decreasing placement.

At a tool boundary, **session interruption** releases the session's GPU allocation while the tool runs; this differs from preempting active inference, which would require recomputation. On return, the session re-enters the waiting pool and is restarted on an eligible node. Physical KV allocation, reuse, and eviction remain the serving engine's responsibility.

## Evidence

The main sweep reports 12 model/workload/concurrency settings: DeepSeek-V4 Flash on two nodes with 4×H100 each, and Qwen3-32B on four nodes with 2×H100 each. Workloads include SWE-bench Lite, Terminal-Bench 2.0, and SWE-verified. Against the best evaluated baseline for each headline metric, the paper reports up to 26.8% lower mean task latency, 36.3% lower mean TTFT, and 29.5% higher decode throughput. These maxima occur in different cells; TAO is not uniformly better on every metric/concurrency combination (for example, Qwen3-32B SWE-Lite latency is 2.5% higher than ThunderAgent at 128 workers).

A separate GLM-4.7 single-node 8×H100 extension reports 15.7–38.4% lower client TTFT and 22.8–24.7% higher decode throughput on SWE-verified over the reported worker settings, relative to ThunderAgent. In a 32-worker DeepSeek SWE-Lite component replay, enabling tool readiness increased decode throughput 4.03% and reduced tool-end-to-first-token delay 10.88%; the duration predictor's mean absolute error was 0.410 s over 10,967 completed tools. Replacing joint DP with best-fit-decreasing in that ablation reduced decode throughput from 1,417.97 to 1,233.72 tok/s.

The paper also tests composition over TokenCake's CPU KV tier on DeepSeek-V4 Flash at 16 and 32 workers. TAO-over-TokenCake raises decode throughput by 12.9% and 4.2% versus consistent-hash routing, with 3.1% and 2.0% lower task latency; TTFT improves 0.7% at 16 workers but is 1.5% higher at 32. Those runs establish serving compatibility, but record no pause/resume/migration events, so they do not attribute the gains to activated KV transfers. A separate fixed-trace InferenceX replay shows 4.2%, 1.8%, and 10.1% higher decode throughput at 16/32/64 workers; cache-aware scheduling has lower TTFT than TAO at 32 and 64 workers.

Scheduler overhead is workload- and load-dependent. The paper reports measured scheduler work below 0.12% for the four-replica Qwen sweep up to 160 workers, but the 64-worker two-replica DeepSeek case reaches 0.297%; high-load Qwen runs also invoke best-fit fallback for 3.06–8.98% of submissions. Overhead accounting excludes fallback execution and failed attempts, so it is explicitly partial.

## Relationship to other systems

[[SGLang]]'s RadixAttention and [[Paged KV Cache]] provide node-local prefix reuse and physical cache management; TAO consumes cache observations as a scheduling signal but does not move cache blocks between nodes. [[HEAR Protocol]] defines a reusable harness–engine message contract, while TAO is a concrete session evaluator and joint scheduler. Its tool-return readiness and capacity-constrained placement are compatible in spirit with HEAR's workflow-intent/engine-state exchange, but the TAO paper does not implement or evaluate HEAR. See [[Tool-Aware Joint Scheduling]] and [[Agentic Serving Scheduling HEAR and TAO]] for the reusable abstraction and comparison.
