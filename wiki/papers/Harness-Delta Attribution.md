# Harness-Delta Attribution: Dissecting Gains from Harness Evolution

**Source:** `raw/papers/2610.22466v1.pdf` (Wenxuan Ding et al., arXiv:2610.22466v1, 2026-10-15)

## Question and protocol

An evolved harness can score better because it learned a reusable procedure, exploited benchmark artifacts, or simply spends more inference-time compute. A single search-set score, and even a held-out score drawn from the same benchmark construction, does not distinguish these explanations. The paper introduces **Harness-Delta Attribution (HDA)** to estimate these sources separately.

Let \(H_0\) be the baseline and \(\hat H\) the evolved harness. HDA measures the observed gain \(\Delta S=S(\hat H)-S(H_0)\), then constructs two controls:

- **Compute-matched baseline** \(H_0^{scale}\): increase the baseline’s inference budget to match the evolved harness. The gain \(T=S(H_0^{scale})-S(H_0)\) estimates what test-time scaling alone can recover.
- **Artifact-neutralized evolved harness** \(\hat H^{neutral}\): minimally disable identified benchmark-specific shortcuts while preserving unrelated edits. The drop \(O=S(\hat H)-S(\hat H^{neutral})\) is a lower bound on detected overfitting.

The residual \(G=S(\hat H^{neutral})-S(H_0^{scale})=\Delta S-O-T\) is an **upper bound** on generalizable improvement, because undetected or entangled shortcuts may remain. HDA separately measures held-out transfer; held-out evaluation alone cannot distinguish added inference compute from a better fixed-budget procedure.

The protocol first requires a paired significance test. If the search-set gain is not significant, it assigns no attribution. Two task-dependent steps receive human review: how to match compute and how to neutralize suspected shortcuts. Thus HDA is a partly automated evaluation workflow, not a fully objective shortcut detector.

## Empirical findings

The authors apply HDA to 25 model–benchmark–framework settings using 11 executor models, three evolution frameworks (Meta-Harness, SkillOpt, and AHE), and four task families: LiveMath, SWE-bench Verified, CREATE, and ALFWorld. Of the 25 settings, 21 show statistically significant search-set gains. Across those 21, detected overfitting accounts for an average 54% of the gain and test-time scaling 14%; their combined attributed share exceeds half in 14 settings.

The dominant source varies by benchmark. ALFWorld and LiveMath gains are often driven by benchmark-specific rules or answer artifacts; CREATE rewards larger diverse answer pools, so extra model calls explain much of some gains; SWE-bench shows clearer residual gains in several settings. All three frameworks independently rediscover a LiveMath multiple-choice shortcut. For example, a meta-option present in the search set is correct in all 21 instances containing it, but only 31% of the other questions in the reported held-out split. This illustrates why nominal transfer within a benchmark can still reflect shared construction artifacts.

## Interpretation and limitations

HDA makes improvement claims more diagnostic by separating mechanisms that a headline score conflates. However, \(O\) is only a lower bound because the audit may miss shortcuts, while \(G\) is only an upper bound because it includes any undetected overfitting. Conclusions depend on the task-specific compute metric and human-reviewed neutralization. Results cover a defined sample of models, frameworks, and benchmarks; they motivate attribution-aware evaluation but do not establish that all harness evolution gains are artifacts.

This protocol sharpens the evaluation questions in [[Agent Self-Improvement]] and adds an evaluation axis to [[Automated Harness Engineering Methods]]. It complements, rather than replaces, held-out transfer tests and the evidence distinctions summarized in [[Harness Evolution Gain Attribution]].
