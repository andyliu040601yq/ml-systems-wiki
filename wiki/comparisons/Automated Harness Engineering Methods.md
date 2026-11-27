# Automated Harness Engineering Methods

**Papers:** [[Meta-Harness]], [[AHE]], [[HarnessX]], [[BOXBOX]], [[Continual Harness]]

These systems all treat some model-external agent structure as an improvement target, but differ in the editable object, how experience is exposed, and whether they also update model parameters. Their scores come from different benchmarks and should not be ranked against one another.

| Method | Search/edit substrate | Feedback and control | Model weights | Main boundary |
| --- | --- | --- | --- | --- |
| Meta-Harness | Task-specific harness programs; coding-agent proposer can inspect and rewrite prior candidates through a filesystem | Scores plus source and full rollout traces; proposer selects what to inspect; Pareto selection can cover accuracy and context cost | Frozen | Terminal-Bench 2 search and final evaluation use the same 89 tasks; stronger held-out evidence comes from text classification and math settings. |
| AHE | Explicit files for prompt, tools, middleware, memory, and other harness components | Layered trace distillation plus versioned edit manifests with predicted effects; ineffective edits can be reverted | Frozen | Coding-agent evaluation is concentrated on Terminal-Bench 2 and SWE-bench Verified; predictions catch fixes better than regressions. |
| HarnessX / AEGIS | Typed, composable processor bundles organized across harness dimensions | Multi-agent trace digestion, planning, typed evolution, critic, and deterministic acceptance; supports variant isolation | Frozen in harness-only evolution; updated by cross-harness GRPO in the co-evolution loop | Broad benchmark suite, but results include noise, global-harness stagnation, and pilot-scale routing experiments. |
| BOXBOX | Executable task harness during long-running software optimization | Agent-triggered edit, validation, and hot reload while preserving task execution state | Frozen in reported harness-revision loop | Focuses on online runtime continuity on PERFRACE tasks. |
| Continual Harness | Prompt, sub-agents, skills, and memory during a continuing embodied task | In-episode refiner plus a slower teacher-assisted training loop | Updated between iterations in co-learning experiments | Pokémon environments; model-training gains depend on frontier-teacher supervision. |

## Cross-cutting synthesis

- **Trace access:** Meta-Harness makes prior runs selectively queryable; AHE adds structured summaries with drill-down to raw traces; HarnessX turns traces and verifier outcomes into inputs to a typed, multi-stage evolution pipeline. The designs suggest a continuum between agent-directed retrieval and explicit evidence processing, not a controlled head-to-head comparison.
- **Auditability:** AHE’s manifest makes each change a falsifiable prediction. HarnessX adds type constraints and deterministic acceptance gates. Meta-Harness leaves diagnosis and edit granularity largely to the proposer, relying on evaluation and inspectable code. BOXBOX addresses continuity and runtime validation during a live task.
- **What “recursive” means:** These methods mostly improve the harness used by a task agent. HarnessX and Continual Harness additionally update model weights in specific co-learning setups. [[AIDE2]] is a distinct case because the outer loop optimizes the research agent’s own code. None of the harness results alone establishes economy-wide recursive AI R&D automation; see [[AI R&D Automation and Intelligence Explosion]].

For when revisions take effect, see [[Online and Offline Harness Improvement]]. For a broader taxonomy of update targets, see [[Agent Self-Improvement]] and [[Recursive Self-Improvement Mechanisms]].
