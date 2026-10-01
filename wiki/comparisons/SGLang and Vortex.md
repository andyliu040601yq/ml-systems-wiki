# SGLang and Vortex

**Papers:** [[SGLang]], [[Vortex]]

These systems address different levels of LLM serving. SGLang exposes the structure of multi-call language-model programs so a runtime can reuse common prompt prefixes and execute branches efficiently. Vortex exposes sparse-attention routing so a runtime can reduce which KV blocks participate in each decode step.

| Question | SGLang | Vortex |
| --- | --- | --- |
| Main target | Multi-call LM programs, including agents, structured output, and multimodal workflows | Decode-time sparse attention algorithms |
| Reuse/selection | RadixAttention shares KV pages for common token prefixes across calls/requests | Indexer selects query-relevant KV blocks for attention |
| Programming interface | Python-embedded prompt DSL (`gen`, `select`, `fork`, `join`) | Python-embedded sparse-attention DSL (`forward_cache`, `forward_indexer`) |
| Runtime contribution | Radix tree with LRU eviction, cache-aware scheduling, compressed FSM decoding, API speculative execution | Paged vTensor lowering, workload planning/fusion, top-k optimization, integration with GQA/MLA decode backends |
| Primary limitation in source | Cache-aware reordering can starve requests; program level is not itself a sparse-attention selector | Focuses on decode, not prefill/training; gains depend on routing overhead and remaining system bottlenecks |

## How the ideas relate

Both depend on physical KV page management, but they use sharing differently. RadixAttention stores one copy of repeated *prefix* KV and shares it among branches and requests. Vortex attends to a *subset* of a request's history according to a dynamic routing score. Prefix sharing reduces duplicate computation and memory across related prompts; sparse routing reduces the history scanned for a particular query. These mechanisms are conceptually compatible, though neither paper reports an implementation combining the two.

The comparison also clarifies that “KV cache optimization” is not one technique. It can mean avoiding recomputation of repeated prefixes, compressing retained state, or reducing the blocks read by attention. The right choice follows the workload's source of redundancy and bottleneck.

## Evidence boundary

SGLang's 2024 experiments and Vortex's 2026 experiments use different models, accelerators, workloads, and baselines. Their speedups should not be numerically ranked against each other.
