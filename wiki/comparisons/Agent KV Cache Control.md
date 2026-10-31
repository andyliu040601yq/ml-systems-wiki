# Agent KV Cache Control: Context Engineering and Cache Engineering

**Sources:** [[Cache Engineering for Agents]], [[HEAR Protocol]], [[SGLang]]

Long-horizon agents need to bound context growth, but “remove history” can mean changing the text supplied to the model or changing which cached states participate in attention. These have different recomputation and correctness consequences.

| Dimension | Text-level context engineering | Cache engineering |
| --- | --- | --- |
| State modified | Logical message/token trajectory | Active KV entries; trajectory remains intact |
| Retained suffix | Editing an earlier prefix invalidates later states and can require re-prefill | Retained states can be reused without recomputation |
| Cache identity | Usually based on token prefix | Must include cache-operation history and positional version |
| Serving support | Summarization/trimming plus ordinary cache handling | Operation-aware radix-tree branches, eviction, and restoration (AKV) |
| Main risk | Lost information or re-prefill cost | Incorrect reuse if history is omitted; retained descendants may still encode dropped context |

The distinction is not simply “context bad, KV good.” `drop` stops selected entries from contributing to future attention, but does not retroactively erase their influence from KV states computed while they were active. Cache engineering therefore gives finer-grained control without text re-prefill, while making cache equivalence and restoration more complex.

## Where coordination fits

[[HEAR Protocol]] addresses how a harness communicates context lifecycle/future-use intent and requirements and how an engine reports cache state and operation outcomes. It is a coordination contract, not itself an eviction policy. AKV, in contrast, defines concrete `append`, `drop`, and `repos` semantics plus serving mechanisms. HEAR could carry controls for a cache-engineering-capable engine, but this composition is not evaluated in the papers.

## Evidence and caveats

The AKV paper's tool-response rolling-drop policy improves average score and reduces cost against its tested threshold-summarization baselines across four benchmarks/models, with serving gains on its particular SGLang-derived deployment. Its ablations show that drop target and reposition matter. Do not generalize that tool responses are always less valuable, that a dropped entry erases its prior influence, or that cache engineering is automatically compatible with every engine. Compare results only with matched model, workload, quality criteria, and serving setup.
