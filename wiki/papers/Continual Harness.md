# Continual Harness: Online Adaptation for Self-Improving Foundation Agents

**Source:** [[raw/papers/2605.09998v1.pdf|Continual Harness paper]] (Karten et al., arXiv:2605.09998v1, May 2026)

**Related:** [[Agent Self-Improvement]], [[Online and Offline Harness Improvement]], [[BOXBOX]]

## Contribution and setting

Continual Harness studies embodied agents playing Pokémon Red and Emerald from a minimal interface: screenshots, an ASCII map of visible tiles, and button inputs. It targets long episodes where environment resets are unavailable or costly. The paper has two related but distinct loops:

1. **Reset-free harness refinement at inference time.** An Agent acts using a harness $H_t=(p,G,K,M)$: system prompt, sub-agents, skills, and memory. Every $F$ steps after warm-up $W$, a Refiner examines the recent trajectory window, diagnoses failures, and applies create/read/update/delete edits to those harness components. The game episode continues without resetting.
2. **Model–harness co-learning across iterations.** An open-source model plays for 256 steps through the live-refining harness. A pairwise process-reward model scores recent transitions; a frontier teacher relabels low-reward windows; soft supervised fine-tuning updates model weights. The saved emulator state carries forward, so the next iteration continues from the same in-game progress while both weights and harness continue to change.

The Agent and Refiner share the same model in the harness-refinement experiments. In the co-learning experiments, however, an open-source student is trained using a frontier-model teacher. The paper does not show that the evaluated open-source models can replace that teacher themselves.

## Harness refinement mechanism

The Refiner identifies patterns such as navigation loops, failed tool calls, stalled objectives, and missed exploration. It then performs four passes: rewrite the system prompt; create, revise, or remove sub-agents; codify successful sequences or repair executable skills; and add, update, or demote memory entries. Earlier failure signatures remain available to later refinement cycles within the episode.

This is in-context/system-state adaptation: the executable harness is edited during a live run, while the model weights stay fixed. The second loop adds weight updates at a slower timescale. This two-timescale design is broader than [[BOXBOX]], whose reported mechanism revises and hot-reloads a task harness but does not train the model weights.

## Results

- On Pokémon Red and Emerald, Continual Harness reduces cumulative button presses versus a minimalist harness and recovers a majority of the measured efficiency gap to a hand-engineered expert harness. The remaining gap is concentrated in dialogue-heavy interiors and multi-turn battle strategy.
- Benefits depend strongly on model capability. On Emerald, Gemini 3 Pro with a from-scratch harness reaches all 31 milestones at a median reported cost of $130, versus 98% at $215 for the minimalist harness. Flash results are high variance; Flash-Lite is below a capability floor, with all tested Continual Harness variants below the minimalist baseline.
- In a pathfinding-skill analysis, the cost deficit relative to a Dijkstra oracle falls from nearly one half to single digits during the run. This provides a direct skill-level measure, in addition to task milestone completion.
- A reset-free co-learning loop produces sustained milestone progress over the reported training iterations for Gemma-4 on Pokémon Red, from both beginning and mid-game checkpoints. The authors report no comparable progress from the untrained baseline in the plotted runs.

These findings support online harness adaptation and a proof of concept for coupled model–harness learning in the tested game setup. They do not establish broad transfer, convergence, or autonomous self-improvement without a teacher: the co-learning loop uses frontier-model relabeling, the open-source students were not capable enough to provide their own teacher, and the experiments do not establish a converged endpoint. The authors leave a matched comparison of reset-free and reset-based training open.

## Relationship to other work

[[BOXBOX]] also lets an agent improve its harness while solving an ongoing task. BOXBOX targets long-running software performance optimization, uses agent-triggered review, and has a kernel validate and hot-reload the new runtime while retaining terminal and workspace context. Continual Harness targets partially observed embodied gameplay and edits prompt, sub-agents, skills, and memory in place; its additional outer loop updates model weights using a teacher and process rewards. The mechanisms overlap at online harness revision but differ in environment, runtime design, and whether model parameters are updated.

See [[Agent Self-Improvement]] for the distinction between harness adaptation and weight learning, and [[Online and Offline Harness Improvement]] for a mechanism-level comparison. The May 2026 paper is arXiv version 1; results here are attributed to that supplied source.
