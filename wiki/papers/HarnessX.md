# HarnessX: A Composable, Adaptive, and Evolvable Agent Harness Foundry

**Source:** `raw/papers/2606.14249v3.pdf` (Darwin Agent Team, arXiv:2606.14249v3, 2026-07-23)

## Architecture

HarnessX treats a runtime harness as a first-class, serializable configuration built from typed **processors** and shared resource slots. Processors consume and emit events, so individual behaviors can be composed or substituted while preserving type contracts. The paper organizes the design space into nine dimensions and argues that this exposes component-level changes that are obscured in monolithic agent code.

AEGIS evolves these configurations from traces and verifier feedback. Its four-stage Digester–Planner–Evolver–Critic pipeline proposes and reviews typed edits; deterministic acceptance gates check candidates before a new harness is committed. The authors describe this as an “operational mirror” of reinforcement learning: harnesses are states, edits are actions, and traced outcomes provide feedback. The analogy motivates defenses against reward hacking, forgetting, and under-exploration; it is symbolic program search, not parameter-gradient training.

HarnessX also describes a coupled loop. The same replay buffer holds reward-annotated trajectories from multiple model and harness versions. AEGIS uses those traces to evolve the harness, while cross-harness GRPO updates model weights. Thus harness evolution provides structural exploration and model training learns finer-grained behavior within those structures. Variant isolation can maintain and route among multiple harnesses when one global configuration regresses on heterogeneous tasks.

## Evidence and boundaries

Across ALFWorld, GAIA, WebShop, \(\tau^3\)-Bench, and SWE-bench Verified, the paper reports average gains of 14.5%, with gains up to 44.0% and larger gains often appearing where the baseline is weaker. The headline results are best interpreted alongside the paper’s diagnostics: in GAIA, a single global harness can stall under heterogeneous task demands, and an apparent peak can fall on identical-config replay. Variant isolation improves that case, while some finer-grained routing experiments are explicitly pilot-scale. The reported outcomes therefore do not establish reliable monotonic improvement on every benchmark.

HarnessX is broader than a harness proposer: it combines compositional runtime abstractions, multi-agent symbolic evolution, and (in its co-evolution experiments) model training. It differs from [[AHE]]’s file-level observability and edit manifests and [[Meta-Harness]]’s proposer-directed filesystem search. It shares harness as the optimization target with both, while [[Continual Harness]] couples harness refinement to teacher-assisted model training in an embodied task. See [[Automated Harness Engineering Methods]] and [[Agent Self-Improvement]].
