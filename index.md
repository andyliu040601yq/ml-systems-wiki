# ML Systems Wiki

## Papers

- [[FlashAttention]] - exact attention optimized for HBM/SRAM traffic through tiling and recomputation.
- [[Vortex]] - programmable, paged-layout-aware sparse attention serving for LLM decode.

## Concepts

- [[Dynamic Sparse Attention]] - query-dependent KV/block routing and its end-to-end serving costs.
- [[IO-Aware Attention]] - FlashAttention's IO-aware exact attention, online softmax, and recomputation.
- [[Paged KV Cache]] - shared physical KV pages, indirection, and prefix-cache-compatible layout.
- [[vTensor and vFlow]] - Vortex's logical sparse-routing language and layout-aware tensor substrate.

## Comparisons

- [[Exact IO-Aware and Sparse Attention]] - FlashAttention's kernel-level IO optimization versus Vortex's programmable dynamic sparse serving.
