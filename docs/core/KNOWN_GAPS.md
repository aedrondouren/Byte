# Known Gaps and Future Work

> Expands on the Known Gaps section of [TECHNICAL_CONCEPT.md](../TECHNICAL_CONCEPT.md#17-known-gaps-and-future-work).
> **Audience:** Technical / Research
> **Prerequisites:** TECHNICAL_CONCEPT.md (full), EVALUATION.md
> **Status:** Complete

This document describes a complete architecture. Several areas are acknowledged as incomplete or requiring further research.

## Macro Discovery (Section 9.1)

The discovery mechanism for identifying reusable execution patterns is an open research problem. The document describes promotion criteria and a general approach (sliding window mining, subgraph matching), but the actual algorithms, equivalence detection strategies, and noise tolerance thresholds are undefined. The system is designed to evolve its discovery mechanism without changing the macro compilation, validation, or execution model. If macro discovery proves intractable, the core thesis still holds through memory and knowledge compression — the efficiency gains would simply be smaller.

Specific gaps within macro discovery:

- **Parameter extraction** — which values from an execution trace become parameters vs. stay fixed? The boundary between generalizable and context-specific values is not yet defined.
- **Hint quality** — how to validate that a Reasoning Hint correctly captures the reasoning context without being too vague (unhelpful for future context injection) or too specific (fails to generalize).
- **Context window mechanics** — the size, content, and injection format of the sliding window of hints and tool results passed to the RPU. How much recent history is useful vs. noisy? Should hints be summarized or passed verbatim?

## Knowledge Extraction Rules (Section 8)

The summarization pipeline describes the flow from situation model to narrative memories to validated knowledge, but the decision criteria for what crosses each threshold are underspecified. The document discusses confidence decay, temporal validity, and revision chains, but the actual extraction rules — what qualifies as knowledge versus preference versus temporary context — require further definition. This will likely emerge from implementation and testing rather than from theoretical design.

### Temporal Pattern Extraction

The mechanism for extracting structured temporal patterns from narrative memory chains (Section 8.2.1) is defined conceptually but not algorithmically. How the summarization pipeline identifies consistent timing from irregular event sequences, how schedule expressions are normalized, and how confidence scores are assigned to extracted patterns all require further specification. The feedback loop (acceptance/denial → knowledge adjustment) is defined, but the initial extraction quality determines whether the loop converges or diverges.

## Cost Model

The scheduler (Section 5) discusses priority, resource quotas, and dynamic repartition, but does not define a cost model for token budgets, inference pricing, storage growth, or retrieval costs. In a system built around minimizing reasoning cost, tracking and optimizing for actual compute expenditure is critical. This will be defined during Phase 1 implementation.

## Failure Recovery

The document discusses replayability and provenance but does not specify the mechanics of recovery when things go wrong mid-execution: tool failures, partially completed plans, inconsistent state after external API changes, or corrupted projections. The event-sourced model makes replay possible, but the decision logic for when to replay, when to abort, and when to escalate requires further design.

## Distributed Conflict Resolution

The architecture is designed around a single-node model with edge-cloud separation. Once multiple edge nodes exist, conflict resolution becomes necessary. The "Git, not blockchain" framing works for a single node, but multi-node scenarios will eventually require some form of conflict resolution strategy — CRDTs, vector clocks, or merge policies. This is deferred until Phase 6.

## EEG and Experimental Modalities (Sections 12, 13)

EEG, EMG, and other biometric inputs are included as potential input modalities. They are experimental extensions, not foundational requirements. The architecture would be functionally unchanged if these modalities were removed. They are included to explore how far the concept can be pushed, not because they are necessary for the core thesis.

## Additional Gaps Identified During Documentation Review

### Perception Uncertainty Propagation

Perception processing is described as deterministic, but the models it uses (object detection, speech-to-text) are probabilistic. The propagation of perception confidence through situation model → intent → execution is not formally specified. Decision thresholds at each stage need definition, particularly for cases where low-confidence perception triggers high-confidence intent.

### Temporal Intent Generator Invariant Tension

The core invariant "AI proposes intent; the kernel executes" is technically violated by the temporal intent generator, which autonomously generates intent from validated knowledge patterns. This is by design — temporal intent is kernel-generated from _validated knowledge_, not AI-generated — but the invariant needs explicit clarification to avoid confusion.

### Cold-Start Problem

The system improves over time through accumulated structure, but the user experience during the initial period (no memory, no knowledge, no macros) is not specified. The bootstrapping process and learning curve need definition.

### Query Complexity

Cross-graph reference resolution mechanics and query cost analysis for common patterns are not specified. The indexing strategy performance characteristics need analysis, particularly for cross-graph queries that traverse multiple domains.

### Compression Thesis Non-Monotonicity

Knowledge decay, macro demotion, and revision chains mean that structure accumulation is not monotonic. The net effect on "accumulated structure" over time — does it still grow despite decay? — needs formal analysis. The concept of "effective structure" (accumulated minus decayed) should be defined.

---

**Related:** [TECHNICAL_CONCEPT.md](../TECHNICAL_CONCEPT.md#17-known-gaps-and-future-work) for the original specification. [EVALUATION.md](EVALUATION.md) for how gaps affect evaluation design. [DESIGN_DECISIONS.md](DESIGN_DECISIONS.md#why-defer-known-gaps) for rationale behind deferring these gaps.
