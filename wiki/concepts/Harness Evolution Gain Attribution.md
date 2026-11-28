# Harness Evolution Gain Attribution

**Source:** [[Harness-Delta Attribution]]

When an agent’s evolved harness scores better, the observed delta can mix at least three effects:

1. **Benchmark overfitting:** the harness exploits task-specific artifacts, answer patterns, or rules in the search data.
2. **Test-time scaling:** the harness performs more inference work, such as additional samples, calls, or turns.
3. **Residual improvement:** performance remains after detected artifacts are neutralized and inference compute is matched.

Harness-Delta Attribution (HDA) estimates these with paired controls. Compare the evolved harness with a compute-matched baseline to estimate the scaling contribution \(T\). Then neutralize detected shortcuts in the evolved harness; the score drop estimates detected overfitting \(O\). The leftover \(G=\Delta S-O-T\) is an upper bound on generalizable improvement, not a direct estimate: undetected shortcuts can remain in the neutralized harness. Likewise, \(O\) is a lower bound on overfitting detected by the audit.

An HDA-style report should state the task objective, score aggregation, inference-cost measure, significance test, compute-matching procedure, identified shortcut evidence, neutralization, and held-out results separately. HDA’s own workflow includes human review of compute matching and shortcut neutralization because both decisions are task-dependent.

The paper’s evaluation across 21 significant settings found detected overfitting and test-time scaling explained an average 54% and 14% of search-set gain, respectively. Those values characterize the sampled Meta-Harness, SkillOpt, and AHE settings; they are not universal rates for harness evolution. Held-out evaluation remains useful, but may not expose artifacts shared across train and test construction, and does not isolate compute scaling by itself.

See [[Harness-Delta Attribution]] for the protocol and examples, [[Automated Harness Engineering Methods]] for the method comparison, and [[Agent Self-Improvement]] for a broader framework for evaluating self-improvement claims.
