# Cache Engineering

**Sources:** [[Cache Engineering for Agents]]

Cache engineering changes the **active KV cache** used by future attention without rewriting the agent's logical token trajectory. It separates semantic participation in attention from physical cache residency: an entry may be dropped from a request's active view while its physical state is still shared or needed elsewhere.

## Operations

| Operation | Effect on active cache | Effect on trajectory/positions |
| --- | --- | --- |
| `append` | Adds KV for new tokens conditioned on the current active cache | Extends logical token history and advances position |
| `drop(P)` | Removes selected KV entries from future attention | Does not edit text, recompute retained KV, or by itself reset positions |
| `repos` | Retains entries but rotates RoPE keys to compact positions | Text is unchanged; subsequent append uses the new compact position range |

The operation sequence matters for cache equivalence. KV for a later token depends on which earlier entries were active when it was computed. Removing an earlier KV later does not undo information already encoded into descendant states. Therefore token-prefix matching alone is not a sound cache identity for this scheme.

## Serving implications

AKV's stateless request API carries full message history plus cache operations; the engine manages state internally. Its extended radix tree branches on drop and reposition operations, drop-aware eviction tracks references from active/restoring requests, and segment-batch attention restores missing state in one prefill pass by splitting attention at operation boundaries while keeping FFN batched.

This differs from [[Paged KV Cache]], which is primarily a physical allocation, sharing, and addressing abstraction. It also differs from [[Dynamic Sparse Attention]]: sparse routing chooses query-dependent blocks for a decode step, whereas cache engineering changes the logical active state across steps. The two may compose but are not equivalent.

## Policy and trade-offs

The paper's rolling-drop policy targets older tool-response KV and compacts RoPE positions, retaining reasoning and recent tool responses. It reports lower cost and improved average benchmark score versus the tested summarization baselines, but this is workload/model-specific evidence. The cache-engineering API makes retention decisions programmable; it does not prove that any particular token role is safely discardable. See [[Agent KV Cache Control]] for the contrast with context engineering and [[HEAR Protocol]] for a complementary harness–engine coordination contract.
