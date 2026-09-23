# Exact IO-Aware and Sparse Attention

**Papers:** [[FlashAttention]], [[Vortex]]

## The relationship

FlashAttention and Vortex optimize different layers of the attention problem. FlashAttention makes a given attention pattern execute efficiently by avoiding HBM materialization; Vortex makes a dynamic sparse pattern expressible and deployable over paged KV caches. Their methods are complementary, not competing algorithms at the same layer.

```text
Sparse-attention serving path

query + paged KV --> router / indexer --> selected block table --> attention backend
                       Vortex's focus          FlashAttention-style IO-aware concern
```

The final backend in Vortex is not the original FlashAttention implementation specifically: the Vortex source names FlashInfer and TensorRT-LLM backends for supported GQA/MLA geometries and adds `cuda_mla`. The relationship above is a systems synthesis: both require turning an abstract attention computation into an IO-efficient kernel/runtime path.

## Comparison

| Question | [[FlashAttention]] | [[Vortex]] |
| --- | --- | --- |
| Primary target | Dense training-time attention on a single GPU | Decode-time dynamic sparse attention in an LLM serving stack |
| Semantic result | Exact dense softmax attention | Exact attention over a dynamically selected subset; selection makes the overall method approximate relative to full attention |
| Main bottleneck addressed | HBM reads/writes for scores/probabilities | Programmability and end-to-end execution of routing over paged KV cache, plus selected-attention execution |
| Core abstraction | SRAM-resident tiles plus online softmax state | vFlow logical program lowered to layout-aware vTensor operators |
| Sparsity | Optional fixed block mask | Static or dynamic patterns; dynamic indexer is a first-class program stage |
| Key trade-off | Recompute intermediates rather than store/read them | Spend work on routing to save KV movement/attention work; optionally approximate top-k |

## Design implication

Sparse attention does not automatically produce serving speedups. It adds a router and indexed KV access, both with their own data movement. FlashAttention supplies the enduring criterion: account for the actual memory hierarchy and intermediate materialization. Vortex applies the same systems mindset at serving level by retaining the runtime's page allocator and using optimized decode backends, while adding a programmable route-selection layer.

Conversely, an IO-efficient dense kernel does not remove the quadratic amount of attended context. When long-generation decode is dominated by KV-cache movement, Vortex's selected blocks can reduce the problem size; the selected-block attention must still be implemented efficiently enough for that saving to survive end to end.

## Boundary of the evidence

The sources evaluate different models, phases, hardware, and baselines. Their numerical speedups should not be compared or multiplied. This page makes an architectural comparison, not a benchmark claim.
