# SGLang: Efficient Execution of Structured Language Model Programs

**Source:** [[raw/papers/paper-sglang_v2.pdf|SGLang paper, arXiv v2]] (Zheng et al., 2024)

**Related:** [[Paged KV Cache]], [[SGLang and Vortex]], [[Vortex]], [[InferenceBench]], [[Agentic Inference Optimization Evaluation]]

## Problem and design

Many LLM applications are programs with multiple dependent or parallel generation calls, shared prompt prefixes, structured outputs, and ordinary control flow. A model-serving API sees individual requests and often misses reuse and parallelism across the larger program. SGLang co-designs a Python-embedded language and runtime so the program structure can guide execution.

The frontend provides prompt-building and generation primitives such as `extend`, `gen`, and `select`, with `fork`/`join` for parallel branches and normal Python control flow. The interpreter represents each prompt as an asynchronous stream: generation work can be enqueued while Python proceeds, and reading a result synchronizes when it is needed. The paper also describes tracing/compiling programs as computation graphs for further optimization; most experiments use interpreter mode.

## RadixAttention: KV reuse across calls

In multi-turn chat, few-shot evaluation, self-consistency, or branch-and-merge reasoning, different calls often share a token prefix. SGLang stores prompt and generated-token KV pages in a radix tree rather than discarding them at the end of each request.

- Edges represent token sequences; prefix lookup finds the longest reusable KV prefix and insertion splits edges as branches diverge.
- KV pages are shared with active requests from the same memory pool. Nodes carry reference counts so pages in use by a running batch cannot be evicted.
- When memory is needed, LRU eviction removes least-recently-used leaves first, preserving shared ancestors for as long as possible.
- The frontend can send a fork-prefix hint; the runtime combines these hints with longest-shared-prefix-first scheduling to increase hits. For the offline batch case, the paper proves DFS order over the request radix tree is optimal when cache capacity is at least the longest request; online scheduling approximates this and can starve requests.

The paper keeps the radix-tree structure on the CPU and reports negligible tree-maintenance overhead. KV pages themselves remain in the GPU memory pool shared with active requests.

The important unit of reuse is a *common prefix*, potentially at multiple levels of a request tree. This complements [[Vortex]]'s vTensor, which makes paged layouts available to sparse operators: RadixAttention organizes *which request prefixes share KV pages*, while Vortex's concern is *how attention routing operates over paged KV*.

## Other runtime optimizations

### Compressed finite-state machines

For regex-constrained generation, SGLang converts the constraint to a finite-state machine and compresses chains of single-outgoing transitions. When the next token sequence is forced by the automaton, the runtime can process several tokens in one model forward pass instead of invoking decoding once per token. The paper reports a 1.6x JSON-decoding throughput improvement; it also reports that rebuilding rather than reusing the FSM per batch makes throughput 2.4x lower.

### API speculative execution

With black-box model APIs, separate `gen` calls can resend the same context and incur repeated latency/input-token charges. SGLang can let an earlier call continue beyond its stop condition, retain surplus generated text, and match it against later generation primitives. The paper describes this as useful when prompts make the expected continuation sufficiently predictable; it is not guaranteed to match every later call.

## Evaluation and evidence

The paper evaluates multi-call program workloads including few-shot tasks, ReAct and generative-agent traces, tree-of-thought, skeleton-of-thought, JSON, multi-turn chat, RAG, and multimodal tasks. Its open-weight experiments use Llama-2 and Mixtral models on A10G/A100 GPUs; the paper compares against vLLM v0.2.5, Guidance v0.1.8, and LMQL v0.7.3. These are workload- and version-specific 2024 results.

- Across tested workloads, SGLang reports up to 6.4x program throughput and up to 3.7x lower latency than the compared systems. Gains are attributed to cache reuse, within-program parallelism, and faster constrained decoding; they depend on workload overlap.
- Cache hit rates range from 50% to 99%, and cache-aware scheduling reaches 96% of the measured optimal hit rate on average in the paper's evaluated workloads.
- With no reuse opportunities on ShareGPT, radix-tree management accounts for 0.2 seconds of a 74.3-second, 100-request run (<0.3%).
- In a month-long Chatbot Arena deployment, the authors report 52.4% and 74.1% RadixAttention hit rates for two served models; one model's first-token latency fell by 1.7x on average.

These figures characterize the workloads and implementation evaluated in the paper; they should not be read as current SGLang-versus-other-engine rankings. [[InferenceBench]] evaluates a later, separate deployment-optimization task and provides a distinct snapshot of default engines under one H100 setup.

## Trade-offs and limits

- Cache-aware reordering improves prefix reuse but can cause starvation; the paper leaves fairness integration as future work.
- Cached KV shares GPU capacity with active requests. Under pressure, cached leaves may be evicted so the scheduler can admit a larger active batch.
- API speculative execution depends on the later call agreeing with surplus output, so prompt design and match rate matter.
- SGLang co-designs runtimes for broad LM programs. Query-dependent selection of KV blocks for sparse attention is the focus of [[Vortex]].

## Takeaway

SGLang exposes program structure to the serving runtime. Its main systems lesson is that cross-call prefix reuse and parallel branches can be optimized only when the runtime sees the larger language-model program, not just isolated generation requests.
