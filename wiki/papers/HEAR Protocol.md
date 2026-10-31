# HEAR: A Bidirectional Harness–Engine Protocol for Agentic LLM Serving

**Source:** [[raw/papers/paper-hear_protocol.pdf|HEAR paper]] (Zhao et al., arXiv:2610.06597v1, 2026)

**Related:** [[Harness–Engine Coordination]], [[Agentic Inference Optimization Evaluation]], [[Cache Engineering]], [[Paged KV Cache]], [[SGLang]], [[InferenceBench]]

## Problem and contribution

An agent harness knows workflow dependencies, agent roles, context lifecycles, and objectives; an inference engine knows request queues, KV residency, resource pressure, supported controls, and actual outcomes. Existing harness–engine interfaces do not provide a shared contract for exchanging this information. This can cause avoidable prefix re-prefill, poor request ordering, and execution-mode choices that ignore workload characteristics.

HEAR (Harness–Engine pAiRing) is a policy-independent protocol for that boundary. It standardizes the meaning and lifecycle of exchanges while leaving workflow selection, engine internals, and optimization policies in their existing layers. Each exchange is associated with the relevant request, context version, or serving instance. HEAR does not prescribe a packet format or require every engine to support every control; capability reports let a harness discover the supported subset.

## Contract and semantics

| Direction | Category | Examples |
| --- | --- | --- |
| Harness → engine | Execution Description and Intent | Agent role/stage, readiness and dependencies, workload shape, context lifecycle, known or predicted future use |
| Harness → engine | Execution Requirements and Control | Scheduling preferences or waiting constraints; cache prepare/retain/release requests; allowed or requested execution mode |
| Engine → harness | State and Capabilities | Prefix residency, queue and resource state, supported operations/configurations, estimated costs |
| Engine → harness | Execution Outcomes | Accepted/completed/rejected/unsupported/failed status, actual mode, cache reuse, latency, resource use and errors |

Four distinctions prevent policy ambiguity: intent is not a control request; a preference is not a hard requirement; observed state is not a reservation or guarantee; and acceptance is not completion. In particular, reporting expected reuse does not itself retain a cache, and an accepted prepare operation does not mean the cache is already ready. A policy can be written abstractly as \(d_i=\pi(h_i,s_j)\), where workflow view \(h_i\) and the latest applicable engine view \(s_j\) jointly inform an execution decision.

## Policy instantiations and results

1. **Online cache-aware runtime coordination.** On SCBench, the harness uses engine-reported reusable-prefix residency to prioritize ready requests. A waiting-protection requirement prevents cache-hot requests from bypassing a request indefinitely. On one RTX PRO 6000 serving Qwen3-8B with a 96-GiB KV cache, Cache-Aware + Guard (40 s) reports 1.61× batch-completion speedup over FCFS, with P50/P95/max TTFT 2.23×/1.45×/1.22× lower and 66.1% reuse versus 19.2% for FCFS. Cache-Aware without Guard raises reuse to 87.6% and shortens the batch further, but its maximum TTFT grows to 224 s, illustrating the fairness/throughput trade-off.
2. **State-conditioned retention.** On a production-derived Mooncake trace, future session-return intent can improve retention at light load, while combining retention with cache-aware scheduling helps at 75% load. At 100% load, no policy dominates: a combined policy can harm session latency and throughput. The paper's evidence is that intent should be conditioned on current engine pressure, not treated as an unconditional pin request.
3. **Role-specific execution configuration.** A selector maps role/workload descriptions to engine-supported, development-profiled modes. It chooses vLLM–H2O for BrowseComp-Plus and OmniKV–H2O for DeepResearchBench, then freezes and verifies the mappings before evaluation. Reported end-to-end speedups are 1.23× and 2.45×, respectively, without observed quality degradation. The winning mapping differs by workload; a universal configuration costs 7.8% on BrowseComp-Plus or 14.0% on DeepResearchBench relative to its selected mapping.

The experiments use replayed workloads or fixed benchmark workflows and specific model/engine/hardware versions. The numbers are evidence for these policy instantiations, not a general guarantee that a protocol alone accelerates serving. HEAR adds a metadata/control path; its cost accounting includes online cache-state updates, scheduling decisions, mode lookup, and outcome reporting in the reported end-to-end runs.

## Systems interpretation and boundaries

HEAR is a control-plane contract, not a cache algorithm or a replacement serving runtime. [[SGLang]] already exposes structured program information and performs prefix reuse; [[Vortex]] provides a page-aware sparse-attention programming substrate. HEAR targets a more general cross-layer semantic boundary that can carry workflow intent and engine feedback to policies like these without standardizing their internal algorithms. [[Cache Engineering]] adds concrete per-request operations over active KV states; those operations could be coordinated through such a contract, but the HEAR paper does not evaluate an integration with AKV.

The protocol intentionally preserves task-selection logic and workflow dependencies. Practical deployments still need authorization, tenant isolation, and bounded waiting for cache controls and workflow metadata; the authors identify these as deployment concerns, not solved guarantees. See [[Harness–Engine Coordination]] for its relationship to other systems and [[Agentic Inference Optimization Evaluation]] for how to interpret its experimental scope.
