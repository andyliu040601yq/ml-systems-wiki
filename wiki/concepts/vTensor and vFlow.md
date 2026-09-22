# vTensor and vFlow

**Source grounded in:** [[Vortex]]

**Related:** [[Paged KV Cache]], [[Dynamic Sparse Attention]]

`vFlow` is Vortex’s Python-embedded DSL for expressing a sparse-attention flow. `vTensor` is its lower-level tensor abstraction that attaches layout semantics to data and makes that flow executable over paged, ragged, and batched serving layouts.

## Separation of concerns

| Layer | Responsibility |
| --- | --- |
| vFlow | Describe KV-only preprocessing and query-time routing from a single-request logical tensor view. |
| Interpreter | Lower each vFlow variable and primitive to vTensor operations. |
| vTensor | Apply sequence-local tensor semantics while propagating ragged shape and physical page layout. |
| Backend | Fuse/schedule operations, compute selected block tables, and invoke optimized sparse decode attention. |

This separation makes an algorithm compositional: a programmer combines GeMM, reductions, elementwise operations, softmax, convolution, persistent `Load`/`Save`, and top-k rather than writing a dedicated serving operator.

## Semantics

A vTensor is \(x_v=(x,C)\), where `x` is the underlying tensor and `C` stores layout metadata. Operators apply a familiar tensor function independently for each sequence in the serving batch, then aggregate outputs into a vTensor with propagated shape/layout. Outputs may have different sequence lengths but must have common non-leading dimensions.

The restriction against transforming the batch dimension is deliberate: it fits decode-time serving, where requests are usually independent while batches are managed by the runtime.

## Why it is useful

The abstraction avoids exposing physical page addresses to a sparse-routing author, but it does not erase the layout. The compiler/runtime can still plan work from actual batch lengths, keep cache state in the page allocator, fuse compatible operator templates, and hand selected blocks directly to paged attention backends. That is the mechanism by which a logical routing algorithm can become an end-to-end serving optimization.
