# ML Systems Wiki Agent Instructions

This repository is my personal knowledge base for studying and researching
machine learning systems.

The purpose of this repository is not simply to summarize papers independently.
The goal is to build and continuously maintain an interconnected knowledge base
that becomes more useful as more sources are added.

The agent is responsible for understanding new sources, extracting important
knowledge, integrating it with existing knowledge, maintaining the wiki structure,
and preserving useful relationships across papers and concepts.


# Repository Structure

## raw/

Contains original source materials.

Structure:

- `raw/papers/`: research papers and technical reports
- `raw/slides/`: lecture slides, talks, and presentations
- `raw/articles/`: technical articles and blog posts
- `raw/docs/`: documentation and technical references
- `raw/notes/`: personal study and research notes

Files under `raw/` are sources of truth.

Never modify, rewrite, rename, or delete files under `raw/` unless explicitly
requested by the user.


## wiki/papers/

Contains structured notes and analyses for individual research papers.

Paper pages should capture the important information necessary to understand
the work from an ML systems research perspective.

Do not mechanically follow a fixed template when it does not fit the paper.


## wiki/concepts/

Contains reusable technical concepts that may appear across multiple papers.

Examples include:

- KV Cache
- Sparse Attention
- PagedAttention
- Continuous Batching
- FlashAttention
- LSH
- GPU Warp
- Triton Program
- Speculative Decoding

Concept pages should synthesize knowledge across sources rather than simply
repeat one paper.

Before creating a new concept page, search the existing wiki to determine
whether an appropriate page already exists.


## wiki/comparisons/

Contains comparisons and higher-level synthesis across papers, systems,
algorithms, and implementations.

Examples:

- MagicPIG vs Block Top-K
- vLLM vs SGLang
- Sparse Attention Methods
- Triton vs CUDA Programming Model

Create comparison pages when the comparison represents useful reusable
knowledge rather than a minor observation.


## index.md

The primary navigation page for the knowledge base.

It should organize and link important:

- papers
- concepts
- comparisons

Read `index.md` before creating new pages.

Update it after meaningful wiki changes.


## log.md

An append-only chronological record of meaningful knowledge-base operations.

Record:

- source ingestion
- major knowledge updates
- new comparisons or synthesis
- maintenance/lint operations

Keep entries concise.


# Automatic Paper Analysis

When ingesting a research paper, independently determine what is important.

Do not require the user to tell you what to pay attention to.

Analyze the work from an ML systems research perspective.

Identify, when relevant:

- research problem
- motivation
- key contributions
- core ideas
- algorithms
- system architecture
- important abstractions
- execution flow
- data flow
- implementation details
- memory management
- GPU/CPU interaction
- kernels and operators
- scheduling
- parallelism
- distributed execution
- training design
- inference design
- serving architecture
- performance optimizations
- hardware/software co-design
- integration with existing ML systems
- experimental methodology
- baselines
- important quantitative results
- ablations
- trade-offs
- limitations
- assumptions
- relationships to existing papers and concepts

Do not mechanically create a section for every item.

Determine which aspects are actually important for understanding the work.

The goal is to understand the source as an ML systems researcher would,
not to produce a generic paper summary.


# Ingest Workflow

When asked to ingest a source:

1. Read the source carefully.

2. Read `index.md`.

3. Inspect relevant existing wiki pages before writing.

4. Determine the source's important technical contributions and concepts.

5. Create or update the appropriate page under `wiki/papers/`.

6. Determine whether important reusable concepts should:
   - update an existing concept page, or
   - justify creation of a new concept page.

7. Integrate the source into existing concept pages when appropriate.

8. Identify relationships with previously ingested work.

9. Add meaningful cross-links between pages.

10. Create or update comparison pages when the relationship between methods
    represents useful reusable knowledge.

11. Update `index.md`.

12. Append a concise entry to `log.md`.

13. Review the resulting changes for correctness, duplication, and consistency.

The goal is integration rather than isolated summarization.


# Knowledge Integration

New sources should improve the existing knowledge base.

When a new source:

- introduces a new concept,
- extends an existing method,
- uses an existing technique differently,
- contradicts another source,
- provides stronger evidence,
- exposes a limitation,
- introduces a new system design,
- or clarifies an existing concept,

update the relevant existing pages.

Do not silently overwrite disagreements between sources.

Preserve important differences and explain which source supports each position.


# Source Grounding

Never invent technical claims.

Distinguish between:

1. information directly supported by sources,
2. synthesis derived from multiple sources,
3. interpretation or reasoning,
4. open questions or uncertainty.

Important technical claims should remain traceable to their source whenever
practical.

Do not present speculation as a fact.

If the source does not provide enough information to support a claim,
state that limitation.


# Wiki Linking

Use Obsidian-style internal links:

[[Sparse Attention]]

[[KV Cache]]

[[Vortex]]

[[MagicPIG]]

Create links when they represent meaningful conceptual relationships.

Do not create unnecessary links merely to increase connectivity.

Prefer linking to an existing canonical concept page rather than creating
duplicate terminology.


# Query Workflow

When answering technical questions using this repository:

1. Read `index.md`.
2. Identify relevant wiki pages.
3. Read those pages.
4. Consult original sources under `raw/` when more evidence or detail is needed.
5. Synthesize the answer from the available knowledge.

If a discussion produces an important reusable insight, comparison, or
conceptual connection, it may be appropriate to integrate that knowledge into
the wiki.

Do not modify the wiki for trivial questions or temporary explanations.


# Wiki Maintenance

When asked to lint or maintain the wiki, inspect for:

- duplicate pages
- duplicate concepts under different names
- missing cross-links
- orphan pages
- contradictions
- stale claims
- poorly structured pages
- important concepts without dedicated pages
- concepts that should be merged
- missing comparisons
- pages that have become too broad
- pages that have become unnecessarily fragmented

Improve organization without changing the meaning of source-supported
knowledge.


# Writing Style

Wiki pages should prioritize technical clarity.

Prefer:

- concise explanations
- precise terminology
- equations when useful
- diagrams or structured descriptions when useful
- explicit relationships between concepts
- concrete system behavior
- important quantitative results

Avoid:

- generic filler
- excessive repetition
- unnecessarily verbose introductions
- unsupported claims
- rewriting the abstract without adding structure or understanding

Paper pages should help the reader understand how the system actually works.


# Git Workflow

After completing one meaningful knowledge-base task:

1. Review all modified and newly created files.

2. Run:

   git status

3. Stage only files relevant to the completed task.

4. Create one concise descriptive commit.

5. Push the commit to:

   origin/main

One meaningful task should normally correspond to one commit.

Example commit messages:

- Add Vortex paper and vTensor concepts
- Add MagicPIG and update sparse attention notes
- Compare MagicPIG and Block Top-K
- Update KV cache concepts from new source
- Reorganize sparse attention knowledge

Never commit:

- API keys
- passwords
- credentials
- `.env` files
- temporary files
- operating-system metadata such as `.DS_Store`

Never use force push.

Never rewrite Git history unless explicitly requested.

If `git push` fails, stop and report the error instead of attempting destructive
Git operations.


# Default Behavior

When the user says:

"Ingest the new paper."

or gives an equivalent instruction:

1. Find the newly added source under `raw/`.
2. Perform the complete ingest workflow.
3. Independently determine what is technically important.
4. Integrate the source with the existing knowledge base.
5. Update `index.md`.
6. Update `log.md`.
7. Review the changes.
8. Commit the completed task.
9. Push it to `origin/main`.

The user should not need to specify what parts of the paper deserve attention.
