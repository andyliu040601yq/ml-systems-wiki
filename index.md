# ML Systems, Agents & RSI Wiki

Research organized across three connected areas: ML systems, agent systems, and
recursive self-improvement. Concepts and comparisons link evidence across them.

## Papers

- [[FlashAttention]] - exact attention optimized for HBM/SRAM traffic through tiling and recomputation.
- [[InferenceBench]] - evaluates open-ended AI-agent optimization of inference servers.
- [[HEAR Protocol]] - a bidirectional harness–engine contract for workflow-aware LLM serving.
- [[Cache Engineering for Agents]] - direct active-KV control and AKV serving for long-horizon agents.
- [[TAO]] - tool-aware joint admission and placement for agentic LLM serving across GPU nodes.
- [[BOXBOX]] - agent-triggered recursive harness revision and runtime hot reloading for long-running software optimization.
- [[Continual Harness]] - reset-free embodied-agent harness adaptation and model–harness co-learning.
- [[ServerBench]] - benchmark for agents designing complete LLM servers against stateful coding workloads and latency-qualified service value.
- [[AIDE2]] - recursive outer-loop optimization of a research agent's code, graded on fixed-budget AI R&D tasks.
- [[Meta-Harness]] - optimizes task harness code by letting a coding-agent proposer inspect prior code, scores, and execution traces.
- [[AHE]] - evolves coding-agent harnesses through component, experience, and decision observability.
- [[HarnessX]] - composes typed harness processors and evolves them with trace-driven AEGIS; also studies harness–model co-evolution.
- [[AI R&D Automation Analysis]] - reviews evidence, conditional feedback-loop scenarios, risks, and policy options for AI R&D automation.
- [[SGLang]] - language/runtime co-design for executing structured, multi-call LLM programs.
- [[Vortex]] - programmable, paged-layout-aware sparse attention serving for LLM decode.

## Concepts

- [[Dynamic Sparse Attention]] - query-dependent KV/block routing and its end-to-end serving costs.
- [[Agentic Inference Optimization Evaluation]] - reliability, integrity, and search-process measures for agentic systems optimization.
- [[Harness–Engine Coordination]] - combining workflow intent with live serving state and outcomes.
- [[Tool-Aware Joint Scheduling]] - combines tool-return readiness, KV locality, and multi-node capacity.
- [[Agent Self-Improvement]] - mechanisms that improve an agent's working procedure during or across tasks, and their evidence boundaries.
- [[AI R&D Automation and Intelligence Explosion]] - distinguishes benchmarked harness/optimizer improvement from broader, uncertain AI R&D feedback scenarios.
- [[Cache Engineering]] - operation-history-aware control of active KV states.
- [[IO-Aware Attention]] - FlashAttention's IO-aware exact attention, online softmax, and recomputation.
- [[Paged KV Cache]] - shared physical KV pages, indirection, and prefix-cache-compatible layout.
- [[vTensor and vFlow]] - Vortex's logical sparse-routing language and layout-aware tensor substrate.

## Comparisons

- [[ServerBench and Agentic Inference Optimization Benchmarks]] - compares constrained algorithm search and end-to-end serving-system design benchmarks.
- [[Agent KV Cache Control]] - text-level context editing versus direct active-KV operations.
- [[Agentic Serving Scheduling HEAR and TAO]] - reusable harness–engine contract versus a concrete joint scheduler.
- [[Online and Offline Harness Improvement]] - task-specific online harness revision versus offline or externally scheduled optimization.
- [[Recursive Self-Improvement Mechanisms]] - distinguishes the level being modified and how recursive gains are evaluated.
- [[Automated Harness Engineering Methods]] - compares harness search substrates, trace feedback, edit controls, and weight updates.
- [[Exact IO-Aware and Sparse Attention]] - FlashAttention's kernel-level IO optimization versus Vortex's programmable dynamic sparse serving.
- [[SGLang and Vortex]] - prefix KV reuse across program calls versus query-dependent sparse KV selection.
