# FlashAttention: Fast and Memory-Efficient Exact Attention with IO-Awareness

**Source:** [[raw/papers/paper-flash_attention.pdf|FlashAttention paper]] (Dao et al., 2022)

**Related:** [[IO-Aware Attention]], [[Dynamic Sparse Attention]], [[Exact IO-Aware and Sparse Attention]], [[Vortex]]

## Central claim

For dense attention, reducing arithmetic complexity is not sufficient to predict wall-clock speed. The original FlashAttention instead computes the *same exact attention* while minimizing traffic between GPU high-bandwidth memory (HBM) and on-chip SRAM. It does not materialize the \(N\times N\) score or probability matrices in HBM, so it reduces attention's extra memory from quadratic to linear in sequence length and can run faster despite recomputing work in the backward pass.

## Algorithm: tile attention around the memory hierarchy

Standard attention materializes

\[
S=QK^T,\quad P=\operatorname{softmax}(S),\quad O=PV,
\]

which writes and rereads \(S,P\in\mathbb{R}^{N\times N}\) in HBM. FlashAttention instead processes blocks of \(K,V\) from SRAM against blocks of \(Q\). For every query row it carries two online softmax statistics: a running maximum \(m\) and normalization sum \(\ell\). After a new score block, it rescales the previous partial output and combines it with the new local contribution under the updated \(m,\ell\).

```text
for each K,V tile loaded to SRAM:
  for each Q tile:
    load Q tile, partial O, m, l to SRAM
    compute local QK^T, row max, exp scores, row sums
    update online m and l; rescale and accumulate O
    write updated O, m, l to HBM
```

Thus matrix multiplication, masking, softmax, dropout, and output accumulation can be fused into a CUDA kernel. The algorithm retains only output plus row-level normalization statistics, not the attention matrix.

## Training: recomputation is an IO optimization

The backward pass would normally consume stored \(S\) and \(P\). FlashAttention saves \(O,m,\ell\) instead, then recomputes score/probability blocks in SRAM while forming gradients. This is selective checkpointing, but its purpose is not merely lower peak memory: the paper reports that additional FLOPs are outweighed by avoided HBM access, so backward can be faster as well.

## IO analysis

Let \(N\) be sequence length, \(d\) head dimension, and \(M\) SRAM capacity, with \(d\leq M\leq Nd\).

| Implementation | HBM accesses in the paper |
| --- | --- |
| Standard materialized attention | \(\Theta(Nd+N^2)\) |
| FlashAttention | \(\Theta(N^2d^2/M)\) |
| Block-sparse FlashAttention, nonzero-block fraction \(s\) | \(\Theta(Nd+N^2d^2s/M)\) |

The paper establishes a lower bound showing no exact-attention algorithm can asymptotically improve on FlashAttention's HBM-access expression across all SRAM sizes in the stated range. This is an IO result, not a claim that dense attention's \(O(N^2d)\) arithmetic is subquadratic.

## Block-sparse extension

With a *predefined block mask*, block-sparse FlashAttention skips zero attention blocks while using the same tiled online-softmax computation for retained blocks. It introduces approximation through the mask and improves the dominant IO term in proportion to its nonzero-block fraction. This differs from [[Dynamic Sparse Attention]], where a query-dependent router must also construct the mask or block table at decode time.

## Reported evidence

All results are reported for the paper's implementations and hardware.

| Setting | Reported result |
| --- | --- |
| GPT-2 attention, A100 | Up to 7.6x faster attention computation than PyTorch; the illustrated forward+backward configuration is 7.3 ms versus 41.7 ms. |
| BERT-large on 8x A100 | 17.4 +/- 1.4 min to target versus 20.0 +/- 1.5 min for the cited MLPerf 1.1 implementation (15% faster). |
| GPT-2 medium on 8x A100 | 6.9 training days versus 21.0 for HuggingFace and 11.5 for Megatron-LM, at comparable reported perplexity. |
| Long Range Arena | FlashAttention is 2.4x faster than the listed standard Transformer; block-sparse FlashAttention is 2.8x faster with similar average accuracy in the reported table. |
| Attention benchmark | Up to 3x faster than the standard PyTorch attention implementation through length 2K; memory footprint scales linearly and is reported up to 20x lower than exact baselines. |
| Long-context capability | Exact FlashAttention reaches 61.4% on Path-X (16K); its block-sparse variant reaches 63.1% on Path-256 (64K). |

## Limits and later systems context

- This 2022 paper targets training attention on a single GPU. It does not solve KV-cache movement, continuous batching, prefix caching, or decode-time dynamic routing.
- Its block-sparse variant assumes a fixed mask. A practical dynamic sparse server must include router/indexing cost and a serving-compatible selected-block representation.
- The original implementation requires a bespoke CUDA kernel for each attention variant and may not transfer across architectures. [[Vortex]] directly addresses an adjacent programmability gap for sparse attention, lowering high-level routing flows to paged tensor operators and delegating selected-block attention to optimized backends.

## Takeaway

FlashAttention changed the optimization target from FLOPs alone to data movement. Its enduring systems lesson is that recomputation and carefully chosen tiling can improve both memory use and latency when they replace slow-memory traffic; sparse attention becomes most useful when built on the same IO-aware execution principle.
