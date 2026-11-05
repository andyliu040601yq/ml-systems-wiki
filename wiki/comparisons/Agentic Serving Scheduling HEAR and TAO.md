# Agentic Serving Scheduling: HEAR and TAO

**Papers:** [[HEAR Protocol]], [[TAO]]

Both papers connect workflow-level facts with inference-engine state, but they make contributions at different layers. HEAR specifies a reusable interaction contract; TAO implements one concrete family of scheduling decisions for tool-interrupted sessions.

| Dimension | HEAR | TAO |
| --- | --- | --- |
| Contribution | Bidirectional message semantics and operation lifecycle between harness and engine | Tool-aware session scoring plus joint multi-node admission and placement |
| Signals | Workflow intent/requirements ↔ cache, queue, capability, and execution outcomes | Tool-return readiness, node-specific decayed cache credit, and token footprint |
| Decision mechanism | Policy-independent; paper instantiates cache-aware ordering/retention and role-specific mode selection | Lexicographic value triplets optimized through a capacity-state DP |
| State/consistency | Distinguishes observations from guarantees and accepted operations from completed ones | Optimizes an asynchronous capacity snapshot, then revalidates against live state; physical KV remains engine-managed |
| Evidence | SCBench/Mooncake coordination and role-specific configurations on two agent benchmarks | Multi-node GPU serving with distributed CPU tools across coding and terminal-agent workloads |

TAO can be viewed as a concrete policy that consumes both harness information (tool progress, session lifecycle, token footprint) and engine information (node-specific cache credit and residual capacity). HEAR could provide common semantics for such exchanges, but TAO does not claim to implement HEAR, and neither paper evaluates the two together. The distinction is useful: a protocol can make policy interfaces portable without selecting a scheduling objective, while a scheduler can be effective in one setting without defining a general cross-engine contract.

Both depend on information freshness and execution feedback. HEAR makes stale observations and operation outcomes explicit protocol concerns; TAO addresses snapshot staleness operationally through asynchronous revalidation. Their results are workload-specific and should not be ranked by headline speedup.
