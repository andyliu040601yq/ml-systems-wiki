# Paged KV Cache

**Source grounded in:** [[Vortex]]

A paged KV cache stores tokens in a shared physical page pool instead of assigning each sequence one contiguous allocation. A logical sequence is reconstructed through a list of physical page IDs, which may be ragged across a batch and can be shared among sequences for prefix caching.

## Why it matters for sparse attention

The layout controls the cost and programmability of decode. A sparse-attention algorithm needs to read selected KV blocks, but those blocks are not necessarily contiguous in physical memory. A conventional tensor program that assumes contiguous batch tensors cannot directly describe or efficiently operate over this indirection.

[[Vortex]] addresses this with [[vTensor and vFlow]]: cache tensors and named auxiliary fields use a paged layout, while its user-facing program operates on a logical sequence view. By using the serving runtime's own paging/allocation mechanism, Vortex retains prefix-cache compatibility rather than copying data into a separate sparse-attention store.

## Layout vocabulary from Vortex

| Layout | Representation |
| --- | --- |
| Batch | Separate \(x_i \in \mathbb{R}^{s_i \times h}\) for each request. |
| Ragged | One contiguous buffer plus pointers that delimit variable-length sequences. |
| Paged | Shared storage plus, for every sequence, a ragged list of page indices identifying its logical blocks. |

In Vortex’s notation, layout metadata is \(C=(b,p,I)\): batch size `b`, sequence offsets `p`, and page-index structure `I`.
