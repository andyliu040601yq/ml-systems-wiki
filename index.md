# ML Systems Wiki

## Papers

- [[FlashAttention]] - exact attention optimized for HBM/SRAM traffic through tiling and recomputation.
- [[InferenceBench]] - evaluates open-ended AI-agent optimization of inference servers.
- [[SGLang]] - language/runtime co-design for executing structured, multi-call LLM programs.
- [[Vortex]] - programmable, paged-layout-aware sparse attention serving for LLM decode.

## Concepts

- [[Dynamic Sparse Attention]] - query-dependent KV/block routing and its end-to-end serving costs.
- [[Agentic Inference Optimization Evaluation]] - reliability, integrity, and search-process measures for agentic systems optimization.
- [[IO-Aware Attention]] - FlashAttention's IO-aware exact attention, online softmax, and recomputation.
- [[Paged KV Cache]] - shared physical KV pages, indirection, and prefix-cache-compatible layout.
- [[vTensor and vFlow]] - Vortex's logical sparse-routing language and layout-aware tensor substrate.

## Comparisons

- [[Agentic Inference Optimization Vortex and InferenceBench]] - DSL-guided algorithm exploration versus open-ended server optimization.
- [[Exact IO-Aware and Sparse Attention]] - FlashAttention's kernel-level IO optimization versus Vortex's programmable dynamic sparse serving.
- [[SGLang and Vortex]] - prefix KV reuse across program calls versus query-dependent sparse KV selection.
