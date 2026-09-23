# Vortex: Efficient and Programmable Sparse Attention Serving for AI Agents

**Source:** [[raw/papers/paper-vortex.pdf|Vortex paper]] (Chen et al., 2026)

**Scope:** decoding-time sparse attention for LLM serving.
**Related:** [[Dynamic Sparse Attention]], [[Paged KV Cache]], [[vTensor and vFlow]], [[FlashAttention]], [[Exact IO-Aware and Sparse Attention]]

## Why this paper matters

Sparse attention can reduce decode-time KV-cache reads, but a useful algorithm is not automatically a useful serving optimization. Modern runtimes store the KV cache in non-contiguous pages, batch requests continuously, and often share pages through prefix caching. Vortex makes new sparse-routing algorithms programmable against a logical per-request tensor view while executing them against those physical layouts and existing optimized attention backends.

The paper's main contribution is therefore an *abstraction-to-serving* path: define cache-side summaries and query-side routing in Python, lower that program to paged tensor operators, fuse and schedule it, then pass its selected block table to a production decode kernel.

## Design

```text
vFlow program (Python)                 Serving runtime
  forward_cache: KV-only summaries       paged KV cache + continuous batch
  forward_indexer: q-dependent scores             |
                 |                                 v
                 +--> interpreter --> vTensor operators --> sparse block table
                                                        |
                                                        v
                                  FlashInfer / TensorRT-LLM / cuda_mla decode
```

### vFlow: program sparse routing in two stages

`vFlow` gives the programmer a batch-size-one, logically contiguous view. A sparse-attention implementation subclasses `vFlow` and separates:

1. `forward_cache(c)`: query-independent, per-block auxiliary state derived from KV cache. For block top-k this is a key centroid per KV block.
2. `forward_indexer(q, c)`: query-dependent routing over blocks, ending in selection such as `topK`. For block top-k: score the cached centroids against `q`, select blocks, and run exact attention only on their tokens.

This decomposition also supports state across decode steps if a named field is declared in the cache stage and updated with `Load`/`Save`, as used to express H2O-style accumulated attention scores. The paper demonstrates the same primitive vocabulary for BlockTopK, DoubleSparse, Quest, H2O, static patterns, and agent-proposed variants.

### vTensor: preserve logical semantics over paged storage

A vTensor is \(x_v=(x,C)\), where `x` is the underlying tensor and layout metadata \(C=(b,p,I)\) records batch size, a sequence pointer array, and page-index structure. An operator applies ordinary tensor semantics independently to every request; Vortex then aggregates outputs and propagates their ragged shape and page layout.

This is the key bridge between a simple single-request program and a continuously batched serving execution:

- queries use normal batch layout;
- KV and named cache fields use the runtime's paged layout;
- temporary results generally use ragged layout;
- sharing the KV paging/allocation machinery retains compatibility with prefix caching.

The abstraction intentionally disallows batch-dimension transformations, matching the mostly sequence-local work in decoding.

### Execution path and optimizations

- At each decode iteration, known batch lengths and layouts drive workload planning. GeMM, elementwise operations, and reductions use a shared chunked template; specialized operators cover softmax and top-k.
- Vortex forms a DAG and greedily fuses compatible operator templates to avoid intermediate reads/writes.
- Routing can make top-k the bottleneck. Its approximate radix top-k can stop before fully refining the threshold bin and sample remaining candidates, yielding a tunable accuracy/speed trade-off. A monotonic score remapping improves radix-bin separation without changing order.
- Vortex uses existing paged decode backends rather than reimplementing attention: FlashInfer or TensorRT-LLM MHA for GQA and TensorRT-LLM MLA where applicable. Its `cuda_mla` kernel fills gaps for general MLA latent geometry and block sizes down to 16.

## Evidence and results

All results below are reported by the paper; they are workload- and platform-specific, not general guarantees.

| Setting | Result |
| --- | --- |
| Qwen3 0.6B–8B on H200, AMC23/AIME24 | Within a 5 pp accuracy budget, block top-k reaches up to 3.46x / 3.60x throughput over full attention; Quest reaches 2.73x / 2.98x. |
| Qwen3-1.7B on H200, 18-hour agent loop | 92 submissions across 23 iterations reached 11,894 tok/s versus 3,437 tok/s for dense attention (3.46x), with AIME24 mean@16 38.96 versus 38.54. |
| GLM-4.7-Flash MLA on one B200 | A rope-aware sparse flow roughly matches full-attention mean@16 (0.752 vs. 0.765) at about 4x throughput; the tighter reported budget reaches 4.7x. |
| MiniMax-M2.7 (229B), TP=4 across four B200s | Block top-k is 1.23x faster at slightly higher mean@16 (0.84 vs. 0.83), and reaches up to 1.37x at a tighter budget. |
| 16K input, 8 req/s | P95 time-per-output-token improves by up to 11.7x for block top-k and 12.8x for Quest. |
| Approximate radix top-k + remapping | At recall@k > 0.97, 1.30–1.62x faster than the radix baseline (1.49x average). |

The reported best agent-generated flows are not evidence that autonomous search discovered a fundamentally new attention mechanism: the paper says its final high-performing algorithms converged to block top-k with algorithmic and systems tuning. The evidence is stronger for the claim that Vortex makes a broad search and realistic end-to-end validation tractable.

## Research observations enabled by the system

- On Qwen3-4B and Qwen3-8B in the reported RULER probes, routing signal is concentrated in two of eight 16-channel groups (`g3` and `g7`). Keeping the four-group half containing both was lossless in that experiment; masking both collapsed tested routing families. This is reported as a Qwen3-family observation, not a universal channel rule.
- For MLA, a routing score that includes the decoupled RoPE component is materially better than one based on compressed content alone. The rope-unaware ablation falls to roughly 0.60–0.63 mean@16 versus about 0.75 for the rope-aware flow.

## Trade-offs and limitations

- The system optimizes **decode**, not prefill, and does not support training/backpropagation.
- Its approximate top-k deliberately relaxes exact selection; the accuracy/recall effect is a parameterized operating-point choice.
- Gains shrink when non-KV costs dominate. At 229B with TP=4, parameter and communication traffic reduce the relative benefit compared with smaller dense models.
- The impressive MLA comparison includes a backend asymmetry noted by the authors: the dense GLM baseline falls back to a slower Triton MLA backend for its unsupported geometry, while Vortex uses `cuda_mla` for sparse flows.

## Takeaway

Vortex recasts sparse-attention serving from “write a custom kernel per algorithm” into a compiler/runtime problem. Its reusable insight is not a new selector alone: it is page-aware, sequence-local tensor composition that lets dynamic routing, cache state, and optimized sparse decode coexist in a modern serving stack. [[FlashAttention]] supplies the adjacent IO-aware kernel principle; see [[Exact IO-Aware and Sparse Attention]] for the boundary and connection.
