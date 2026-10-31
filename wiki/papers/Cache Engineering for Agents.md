# Cache Engineering for Agents

**Source:** [[raw/papers/paper-cache_engineering_for_agents.pdf|Cache Engineering paper]] (Fu et al., arXiv:2610.28577v1, 2026)

**Related:** [[Cache Engineering]], [[Agent KV Cache Control]], [[Paged KV Cache]], [[SGLang]], [[HEAR Protocol]], [[Harness–Engine Coordination]]

## Motivation

Long-horizon agents accumulate tool responses and reasoning in their logical trajectory. Conventional context engineering trims or summarizes text, but editing an earlier prefix invalidates subsequent states and requires re-prefilling the retained suffix. Retaining everything instead increases KV I/O/storage and can exceed a model's useful context range. This paper proposes **cache engineering**: control which KV entries participate in future attention while leaving the logical token history intact.

This is not ordinary physical cache eviction. The active cache is the logical set of KV states visible to attention; the serving engine separately manages physical residency and may restore evicted states later.

## Cache operations and their semantics

- `append(x)` performs ordinary autoregressive cache growth, conditioning new states on the currently active cache.
- `drop(P)` excludes selected logical-token KV entries from subsequent attention but leaves the text trajectory, retained entries, and their positions unchanged.
- `repos` re-encodes retained keys at compacted RoPE positions and resets the next append position. Values remain unchanged; for a key moved from position \(p\) to \(q\), the transformation is \(K' = R(q)R(p)^{-1}K\).

Operation order is semantically significant. A retained token's KV may have been computed while different earlier cache entries were active. Thus, identical token prefixes and retained positions do not necessarily imply equivalent states. Dropping an entry later does not erase information already incorporated into descendants' hidden states. The complete token-plus-operation history is needed to establish correct reuse.

## AKV serving design

AKV follows **stateless interface, stateful serving**: a client sends the full logical messages plus `drop`/`reposition` instructions; the server internally tracks cache identity, reuse, eviction, and restoration. Three mechanisms address the resulting systems problems:

1. **Extended radix tree:** drop nodes record where the history branches and which logical cache entries were removed. Prefix matching then distinguishes states produced by different operation histories. Reposition nodes distinguish positional versions after RoPE changes.
2. **Operation-aware eviction and isolation:** a KV entry becomes evictable only when no active request or restoration needs it. Dropped cache is preferred for reclaiming memory; placeholders in the tree preserve paths to reusable descendants. Independent positional versions can be evicted separately.
3. **Segment-batch attention:** missing history-dependent states are restored in a single prefill pass. Queries split at drop/reposition boundaries so each segment sees the active cache and original position encoding appropriate to its history. Only attention is segmented; FFN processes the full sequence in one batch, avoiding repeated FFN weight reads.

## Policy and evidence

The main policy retains reasoning and recent tool responses while incrementally dropping older tool-response KV (latest 12 tool responses in the reported serving setup), then uses `repos` at the normal summarization threshold to control position growth. Across three models and four agent benchmarks, the paper reports average score gains of 1.3–2.1 points and 27.7% lower average inference cost than threshold-triggered Standard summarization; versus Text Engineering, average cost is 21.9% lower. A held-out recursive self-improvement policy reduces cost by a further 7.1% versus expert design while preserving the baseline accuracy criterion.

For GPT-OSS-120B on the paper's SWE-Verified serving setup, AKV at its peak-throughput concurrency achieves 1.51× lower P95 request latency and 1.18× output throughput than Standard at its peak-throughput setting. At concurrency 14, it reports 9.09 s mean / 28.49 s P95 request latency and 369 output tokens/s, versus Standard's peak-throughput concurrency 12 with 12.39 s / 43.16 s and 313 output tokens/s. The comparisons are setup-specific; AKV's advantage widens at high concurrency when standard cache pressure causes more re-prefill.

In an ablation, adding reposition raises BrowseComp-Plus pass rate for tool-drop from 55% to 62%; dropping reasoning KV instead of tool-response KV reduces SWE performance in the tested configuration. These results support the tested policy, not a universal claim that tool responses are always safe to remove. The paper notes further policy exploration, training agents under cache engineering, and support for linear-attention models as future work.

## Relationship to other cache techniques

[[SGLang]]/RadixAttention reuses equivalent shared token prefixes, while AKV extends prefix identity to include cache-operation history and positional versions. [[Paged KV Cache]] describes physical indirection and page sharing; it does not, by itself, encode whether two same-token prefixes produced equivalent active states. [[Vortex]] selects a subset of blocks per query for sparse attention; cache engineering changes the active cache across workflow steps. These mechanisms act at different semantic layers and may be complementary, but the paper does not report a combined implementation.

[[HEAR Protocol]] is also complementary: HEAR defines how harness intent, requirements, engine state, and outcomes can be exchanged, whereas cache engineering defines operations and an engine design for applying those operations. An integration is a plausible design connection, not an evaluated result in either paper. See [[Agent KV Cache Control]] for the broader comparison with text-level context editing.
