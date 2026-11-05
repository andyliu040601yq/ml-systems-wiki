# Harness–Engine Coordination

**Sources:** [[HEAR Protocol]], [[Cache Engineering for Agents]], [[TAO]], [[InferenceBench]], [[Vortex]]

An agent harness sees workflow dependencies, roles, context lifecycles, and application objectives. An inference engine sees queues, cache residency, supported operations, and resource pressure. Coordination is the systems problem of making an execution decision from the relevant parts of both views while preserving workflow semantics.

## HEAR's reusable contract

[[HEAR Protocol]] separates the interaction contract from policy. It distinguishes four message categories—workflow intent, explicit requirements/controls, engine state/capabilities, and realized outcomes—and clarifies intent vs. control, preference vs. requirement, observation vs. guarantee, and acceptance vs. completion. Its example policies use the same contract at different timescales: cache-aware scheduling/retention online and workload-specific mode selection from calibrated profiles.

This complements [[SGLang]]'s program-aware execution and prefix reuse, [[Vortex]]'s programmable sparse-attention substrate, and [[Cache Engineering]]'s explicit operations over active KV state. HEAR is not a replacement for these mechanisms; it could provide common semantics for connecting harness-side workflow knowledge to supported engine controls. The proposed combination remains an integration hypothesis, not a result reported by these papers.

[[TAO]] is a concrete scheduling policy rather than a general protocol: it combines tool-return readiness, node-specific KV-cache credit, and token footprint, then jointly admits and places waiting sessions under multi-node capacity constraints. It illustrates the kind of policy that can use both harness and engine views. The TAO paper does not implement HEAR; see [[Agentic Serving Scheduling HEAR and TAO]] for the distinction.

## Why both views and the feedback loop matter

- Harness future-use intent alone cannot guarantee a context remains resident; the engine must report current state and confirm the realized result.
- Engine cache state alone does not reveal workflow dependencies or how long a request can wait; harness constraints can prevent a throughput-oriented scheduler from starving cold requests.
- Stable workload shape and role can guide engine-mode selection, but the selected mode should be measured and quality-checked for that workload rather than applied universally.
- State is time-sensitive: a residency observation can become stale before dispatch. Correct coordination therefore needs outcome feedback and policies that tolerate state changes.

## Evidence boundary

HEAR's SCBench, Mooncake, BrowseComp-Plus, and DeepResearchBench results demonstrate two specific policy instantiations. [[InferenceBench]] evaluates the broader agent task of finding, validating, and preserving an optimized server; [[Vortex]] evaluates agent search in a constrained algorithm substrate. These answer different questions and should not be ranked by headline speedup alone. See [[Agentic Inference Optimization Evaluation]].
