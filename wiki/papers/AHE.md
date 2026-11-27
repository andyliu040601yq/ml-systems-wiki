# Agentic Harness Engineering (AHE)

**Full title:** *Agentic Harness Engineering: Observability-Driven Automatic Evolution of Coding-Agent Harnesses*
**Source:** `raw/papers/2604.25850v5.pdf` (Jiahang Lin et al., arXiv:2604.25850v5, 2026-11-22)

## Design

AHE evolves an editable coding-agent harness through three forms of observability:

1. **Component observability:** prompt, tools, middleware, sub-agents, memory, and related components are exposed as separate files, making edits explicit and revertible.
2. **Experience observability:** an Agent Debugger distills large rollout traces into a layered evidence corpus (benchmark overview, per-task reports, and drill-down access to raw traces) so the evolution agent can investigate causes without loading every token at once.
3. **Decision observability:** every edit is recorded in a versioned manifest with a predicted task-level effect. Subsequent outcomes can confirm the prediction or trigger rollback.

The Evolve Agent reads the evidence, changes harness files, and records its rationale. Its workspace is bounded: evaluation data, tracer, verifier, and model configuration are read-only. The design makes changes auditable and testable, while keeping evaluation infrastructure outside the editable action space.

## Results

On Terminal-Bench 2 (89 tasks), ten iterations raise pass@1 from 69.7% to 77.0%, above the paper’s Codex (71.9%), ACE (68.9%), and Training-Free GRPO (72.3%) baselines. The frozen harness transfers to SWE-bench Verified: 75.6% success over 500 tasks, using 461k mean tokens per trial versus 526k for the NexAU 0 seed. When evaluated on Terminal-Bench 2 without further evolution, the same harness also improves three alternate model families by 5.1–10.1 percentage points.

Component ablations attribute more of the measured gain to tools, middleware, and long-term memory than to the system prompt. Those effects do not add linearly: component interactions can make individually helpful edits redundant or harmful. AHE’s own predictions are better at identifying fixes than forecasting regressions, leaving regression foresight as a weakness.

## Interpretation and limitations

AHE is a controlled, offline harness-evolution loop. It changes model-external code and state-management components, not model weights. Its observability apparatus gives a concrete mechanism for turning trajectory evidence into falsifiable engineering hypotheses; the manifest is not proof of causal attribution by itself. The paper reports transfer beyond its evolution target, but the evaluation still covers a limited set of coding benchmarks and model operating points. The authors explicitly describe it as a research prototype rather than a mature autonomous self-improvement system.

Compare its layered evidence and prediction manifests with [[Meta-Harness]]’s filesystem-mediated trace access and [[HarnessX]]’s typed components plus trace-driven AEGIS. The broader taxonomy is in [[Agent Self-Improvement]] and [[Automated Harness Engineering Methods]].
