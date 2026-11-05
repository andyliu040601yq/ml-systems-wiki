# ML Systems Wiki

## Papers

- [[FlashAttention]] - exact attention optimized for HBM/SRAM traffic through tiling and recomputation.
- [[InferenceBench]] - evaluates open-ended AI-agent optimization of inference servers.
- [[HEAR Protocol]] - a bidirectional harness–engine contract for workflow-aware LLM serving.
- [[Cache Engineering for Agents]] - direct active-KV control and AKV serving for long-horizon agents.
- [[TAO]] - tool-aware joint admission and placement for agentic LLM serving across GPU nodes.
- [[SGLang]] - language/runtime co-design for executing structured, multi-call LLM programs.
- [[Vortex]] - programmable, paged-layout-aware sparse attention serving for LLM decode.

## Concepts

- [[Dynamic Sparse Attention]] - query-dependent KV/block routing and its end-to-end serving costs.
- [[Agentic Inference Optimization Evaluation]] - reliability, integrity, and search-process measures for agentic systems optimization.
- [[Harness–Engine Coordination]] - combining workflow intent with live serving state and outcomes.
- [[Tool-Aware Joint Scheduling]] - combines tool-return readiness, KV locality, and multi-node capacity.
- [[Cache Engineering]] - operation-history-aware control of active KV states.
- [[IO-Aware Attention]] - FlashAttention's IO-aware exact attention, online softmax, and recomputation.
- [[Paged KV Cache]] - shared physical KV pages, indirection, and prefix-cache-compatible layout.
- [[vTensor and vFlow]] - Vortex's logical sparse-routing language and layout-aware tensor substrate.

## Comparisons

- [[Agentic Inference Optimization Vortex and InferenceBench]] - DSL-guided algorithm exploration versus open-ended server optimization.
- [[Agent KV Cache Control]] - text-level context editing versus direct active-KV operations.
- [[Agentic Serving Scheduling HEAR and TAO]] - reusable harness–engine contract versus a concrete joint scheduler.
- [[Exact IO-Aware and Sparse Attention]] - FlashAttention's kernel-level IO optimization versus Vortex's programmable dynamic sparse serving.
- [[SGLang and Vortex]] - prefix KV reuse across program calls versus query-dependent sparse KV selection.
