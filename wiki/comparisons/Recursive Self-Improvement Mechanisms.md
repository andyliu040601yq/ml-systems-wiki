# Recursive Self-Improvement Mechanisms

**Papers:** [[AIDE2]], [[Continual Harness]], [[BOXBOX]], [[Cache Engineering for Agents]]

“Self-improvement” can refer to changes at different levels of an agent system. These papers differ in what is updated, how feedback selects changes, and when updates take effect. Their reported scores use different benchmarks and are not directly comparable.

| System | What changes? | Update loop and evidence | Important boundary |
| --- | --- | --- | --- |
| AIDE2 | Research-agent code, including search policy, context management, and robustness mechanisms | Outer-loop agent proposes rewrites; inner agent is graded on private held-out AI R&D tasks under fixed dollar budgets. Seven rewrites were accepted in the reported eight-day run; AIDE85 transferred to four external benchmarks. | Ignition test had three seeds per arm and was inconclusive about whether AIDE47 is a better self-improver than AIDEhuman. Model weights remain fixed during the RSI run. |
| Continual Harness | Prompt, sub-agents, skills, and memory during an embodied task; additionally, student model weights between training iterations | In-episode Refiner edits harness state without resetting Pokémon gameplay. A slower co-learning loop uses PRM scores and frontier-teacher relabeling for soft SFT. | Evidence is in Pokémon Red/Emerald; gains depend on model capability, and the student relies on an external frontier teacher. |
| BOXBOX | Full executable task harness during long-running software optimization | Task agent triggers a review; kernel validates and hot-reloads the revision while preserving execution context. | Changes the harness, not model weights; evaluated on PERFRACE software-performance tasks. |
| Cache Engineering for Agents | Active KV-cache policy for future attention and a held-out recursive policy design | Policy is evaluated on held-out agent tasks; serving system tracks operation history and cache-state equivalence. | Changes cache participation and serving behavior, not the harness or model weights. |

## Reading the evidence

The broadest recursion level here is AIDE2: the object being optimized is the procedure that performs research optimization. Its private selection grade separates the inner agent's public search feedback from the outer loop's candidate-selection signal, and fixed budgets constrain apparent gains from extra compute. This reduces direct metric gaming but does not remove noise from multi-task inner search and outer grading.

Continual Harness couples two time scales: harness edits occur within one continuing environment episode, while model training happens between iterations. BOXBOX focuses on safe runtime replacement and state continuity during one task. Cache Engineering changes an inference-time resource policy. The label RSI alone therefore says little about whether a system learns reusable skills, changes parameters, adapts tools, or alters memory behavior; the target and evaluation protocol must be stated explicitly.

For the separate comparison of fixed, offline, between-attempt, and in-task harness refinement, see [[Online and Offline Harness Improvement]]. For AIDE2's two-level procedure and results, see [[AIDE2]]; the broader taxonomy is [[Agent Self-Improvement]].
