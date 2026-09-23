# IO-Aware Attention

**Source grounded in:** [[FlashAttention]]

[[FlashAttention]] is an exact attention algorithm and fused GPU implementation that is IO-aware: it organizes attention around the HBM-to-SRAM hierarchy rather than materializing the full score and probability matrices in HBM.

## The principle

Dense attention still has \(O(N^2d)\) arithmetic. FlashAttention improves its realized performance and memory footprint by reducing slow-memory reads/writes. Tiling computes blocks of \(QK^T\) in SRAM, while online numerically stable softmax statistics let each query row accumulate a correct output without holding all of its scores simultaneously.

The backward pass stores output and per-row softmax normalization statistics, then recomputes score/probability tiles in SRAM. This is a key systems trade: more arithmetic can be beneficial when it avoids much more expensive HBM traffic.

## What it does and does not optimize

| Dimension | [[FlashAttention]] | [[Dynamic Sparse Attention]] |
| --- | --- | --- |
| Attention result | Exact dense attention | Typically approximate, via selected KV blocks/tokens |
| Primary mechanism | IO-aware tiling, fusion, recomputation | Reduce the attended KV set; compute a routing decision |
| Asymptotic dense FLOPs | Remains quadratic | Can reduce attended attention work |
| End-to-end concern | HBM/SRAM traffic during attention | Router cost, cache layout, selected-attention backend, quality |

These techniques compose: [[FlashAttention]] includes a block-sparse extension for a fixed block mask, and [[Vortex]] uses optimized paged attention backends after dynamically constructing sparse block tables. See [[Exact IO-Aware and Sparse Attention]] for the synthesis.

## Reusable design pattern

1. Identify the large intermediate that creates slow-memory traffic.
2. Tile computation so active input/output state fits fast memory.
3. Carry sufficient online statistics to preserve exact semantics across tiles.
4. Save compact state and recompute local intermediates in backward if this replaces expensive reads.
5. Fuse the resulting pipeline, and measure wall-clock/IO rather than FLOPs alone.
