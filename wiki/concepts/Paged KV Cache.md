# Paged KV Cache

**Source grounded in:** [[SGLang]], [[Vortex]], [[Cache Engineering for Agents]]

A paged KV cache stores tokens in a shared physical page pool instead of assigning each sequence one contiguous allocation. A logical sequence is reconstructed through a list of physical page IDs, which may be ragged across a batch and can be shared among sequences for prefix caching.

## Prefix sharing and cache residency

[[SGLang]]'s RadixAttention indexes token-prefix paths in a radix tree and maps them to KV pages. Multiple requests can reuse pages for their longest common prefix. It keeps cached pages in the active KV memory pool, uses reference counts to protect pages used by active requests, and evicts least-recently-used leaves first. Prefix-aware scheduling increases reuse, with a starvation trade-off under greedy reordering.

In the SGLang paper's layout, each page holds one token. Other runtimes may choose different page granularities; the general property is indirection from logical token order to physical storage.

## Why it matters for sparse attention

The layout controls the cost and programmability of decode. A sparse-attention algorithm needs to read selected KV blocks, but those blocks are not necessarily contiguous in physical memory. A conventional tensor program that assumes contiguous batch tensors cannot directly describe or efficiently operate over this indirection.

[[Vortex]] addresses this with [[vTensor and vFlow]]: cache tensors and named auxiliary fields use a paged layout, while its user-facing program operates on a logical sequence view. By using the serving runtime's own paging/allocation mechanism, Vortex retains compatibility with prefix caching rather than copying data into a separate sparse-attention store. The two papers thus emphasize separate operations on the same page substrate: SGLang shares repeated prefixes across calls, while Vortex selects blocks within a request for sparse attention.

## Cache-operation history is a separate axis

[[Cache Engineering for Agents]] shows that physical paging and prefix matching are not enough when an agent can drop active KV or reposition keys. Two requests with the same retained token sequence may have different cache states because later KV was computed under different active-cache histories. AKV extends its radix tree with operation-history and position-version branches, while the serving engine separately manages physical residency and eviction. This is orthogonal to page size and address indirection: paging says where KV lives; operation history says whether a state is semantically equivalent and reusable.

## Layout vocabulary from Vortex

| Layout | Representation |
| --- | --- |
| Batch | Separate \(x_i \in \mathbb{R}^{s_i \times h}\) for each request. |
| Ragged | One contiguous buffer plus pointers that delimit variable-length sequences. |
| Paged | Shared storage plus, for every sequence, a ragged list of page indices identifying its logical blocks. |

In Vortex’s notation, layout metadata is \(C=(b,p,I)\): batch size `b`, sequence offsets `p`, and page-index structure `I`.
