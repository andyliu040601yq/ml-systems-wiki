# Meta-Harness: End-to-End Optimization of Model Harnesses

**Source:** `raw/papers/2603.28052v1.pdf` (Yoonho Lee et al., arXiv:2603.28052v1, 2026-03-30)

## Problem and central idea

An LLM application’s harness determines what information is stored, retrieved, and presented to a fixed model. Meta-Harness automates search over this task-specific code. Its proposer is itself a coding agent: it can inspect and edit prior harnesses and selectively query a filesystem containing source, scores, and full rollout traces. The paper’s main design claim is that access to diagnostic experience preserves credit-assignment information that scalar rewards or short summaries lose.

For fixed model \(M\) and task distribution \(X\), the objective is to find a harness \(H\) maximizing expected task reward. With multiple objectives, such as accuracy and context cost, the search returns a Pareto frontier. Each iteration evaluates proposed programs, validates their interfaces, and adds their code and execution evidence to the shared filesystem. A typical run evaluates about 60 harnesses over 20 iterations; the underlying task model remains frozen.

## Evidence

- **Online text classification:** Meta-Harness reports a 7.7-point gain over ACE while using about 4x fewer additional context tokens. On nine previously unseen datasets, it reports a 2.9-point average advantage over the next best method. In an interface ablation, full access to traces reaches median/best accuracy of 50.0/56.7, versus 34.6/41.3 with scores only and 34.9/38.7 with scores plus summaries.
- **Retrieval-augmented math:** a discovered harness improves accuracy by an average 4.7 points on 200 IMO-level problems across five held-out models.
- **Agentic coding:** discovered harnesses outperform reported hand-engineered Terminal-Bench 2 agents, including Terminus-KIRA. The paper explicitly treats the same 89-task benchmark as a discovery problem: search and final evaluation share it. Manual and regex audits check for task-specific leakage, but this result is weaker evidence of generalization than a held-out coding benchmark.

The proposer uses Claude Code with Opus 4.6 in the reported runs. The agent chooses which artifacts to inspect and can make local edits or larger rewrites; the outer loop does not prescribe a fixed mutation operator.

## Systems interpretation and limits

Meta-Harness improves the **harness search procedure** through accumulated executable experience; it does not update the underlying model weights, and the proposer itself is not the object of recursive optimization in these experiments. This places it near [[AHE]] and [[HarnessX]] as automated harness engineering, while [[AIDE2]] targets the research optimizer itself. See [[Agent Self-Improvement]] and [[Automated Harness Engineering Methods]].

The strongest evidence for the proposed interface comes from the text-classification ablation. The coding result is benchmark-specific by design, and the paper notes that code-space overfitting remains possible even if brittle hard-coding is easier to inspect than weight-space overfitting. The reported results support selective access to traces as a useful search interface; they do not establish that every domain benefits equally or that raw traces alone guarantee causal attribution.
