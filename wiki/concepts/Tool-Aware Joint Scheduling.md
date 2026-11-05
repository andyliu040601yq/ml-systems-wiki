# Tool-Aware Joint Scheduling

**Source:** [[TAO]]

Agent-serving demand is intermittent: a session waits on an external tool, then returns for another model turn with a context that may be cached on one GPU node but not another. A scheduler must balance **when inference will be needed**, **where useful KV state exists**, and **which set of sessions fits**. These are related signals, not independent per-session priorities.

## TAO's value interface

For each eligible session-node pair, TAO uses a lexicographic triplet $(k_{i,b},r_i,t_i)$:

1. $k_{i,b}$, the node-specific cache-credit estimate, decays with observation age (evaluated half-life: 10 seconds).
2. $r_i$, near-term tool-return readiness, estimates whether inference will be needed within a short horizon (1 second in the reported configuration).
3. $t_i$, token footprint, breaks ties after cache credit and readiness while also contributing to capacity cost.

Lexicographic precedence makes the policy explicit: it does not trade a small amount of higher-priority cache credit for an arbitrary weighted amount of readiness. This is a choice made by TAO, not an inherent ordering for every workload.

## Joint admission and placement

Given a waiting pool and a snapshot of residual node capacities, TAO runs a dynamic program over a discretized multiple-knapsack/generalized-assignment state to jointly choose sessions and destination nodes. It handles node-specific values, so the optimal set depends on assignments as a whole, not only each session's scalar score. The DP is exact for the frozen discretized problem; quantization can exclude a solution feasible in the undiscretized problem. It runs asynchronously and the executor revalidates before applying results. If its state budget is exhausted, best-fit-decreasing is the fallback.

Interruption occurs at a tool boundary: the session releases inference-node resources while the CPU tool executes, then returns to the waiting pool. This avoids preempting active GPU inference and its recomputation cost. TAO coordinates admission and placement; the serving engine retains authority over physical KV allocation, reuse, and eviction.

## Trade-offs and evidence boundaries

Tool timing helps only when predictions are useful and there is a choice to make. Cache credit is a scheduler-side estimate, not a reservation; node-local cache may be evicted before dispatch. A binary readiness horizon also simplifies a noisy, continuous return-time estimate. Finally, exactness applies to the discretized snapshot, not the asynchronous live cluster. Benchmarks report gains across many but not all metric/load cells, and fallback overhead is not fully accounted for. See [[TAO]] for setups/results and [[Harness–Engine Coordination]] for the broader cross-layer context.
