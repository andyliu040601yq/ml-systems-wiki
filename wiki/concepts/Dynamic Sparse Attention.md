# Dynamic Sparse Attention

**Source grounded in:** [[Vortex]]

Dynamic sparse attention constructs the set of KV entries or blocks to attend to from the current query, hidden state, or other input-dependent state. This differs from static sparse attention, whose layout follows a fixed positional pattern independent of content.

## Decode-time pipeline

```text
KV cache --> query-independent summaries --> current query scores summaries
                                                  |
                                                  v
                                      select blocks / tokens --> exact attention on selection
```

The routing stage is part of the algorithm, not just an attention-kernel input. A practical dynamic method must account for its score computation, reduction, selection, data-layout overhead, and the selected-attention kernel together.

## In Vortex

[[Vortex]] expresses this pipeline as `forward_cache` (summaries independent of the query) and `forward_indexer` (query-dependent scoring and selection). The block-top-k example caches a key centroid for each block, scores centroids against the current query, selects top-k blocks, then applies exact softmax attention to all tokens in those blocks.

The paper uses the same model to express Quest (per-feature key envelopes), DoubleSparse, H2O-style persistent score state, and static layouts. See [[vTensor and vFlow]] for how the logical program runs over a [[Paged KV Cache]].

## Systems implications

- Selecting less KV data helps only if routing plus attention is faster end to end than dense decode.
- Router state can be query-independent cache data or persistent cross-step statistics; both must fit the serving runtime's allocation and sharing behavior.
- Block granularity trades selection precision against metadata/routing cost and kernel efficiency. Vortex reports block size 16 with a moderate budget of about 125 blocks as a robust reported default across its evaluated settings, not a universal optimum.
- Approximate selection can shift the throughput/quality frontier. Vortex’s early-terminated radix top-k targets a recall operating point rather than exact top-k equivalence.

## Open boundary

The Vortex source treats decode as the target because growing KV reads dominate its long-generation workloads. Sparse prefill and training require different execution and gradient considerations and remain outside its implementation.
