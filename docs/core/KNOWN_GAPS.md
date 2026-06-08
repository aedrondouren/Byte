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

### Hierarchical Macro Discovery

Skill-derived macro discovery adds a structured seed to the discovery problem but introduces its own gaps:

- **Skill-to-macro transformation quality** — how to measure when a derived macro is truly equivalent to its source skill's intent, not just the observed trace. The skill's instructions define intent; the trace captures one execution. The gap between intent and trace is the overfitting risk.
- **Parameter extraction from skill templates** — skill instruction files contain configurable parameters (tool selections, prompt variables, behavioral thresholds). Which of these become macro parameters vs. fixed values? The boundary depends on how the user actually uses the skill, which is not known at skill-install time.
- **Hierarchical macro discovery algorithms** — how to distinguish a composition pattern (macro A + macro B always invoked together) from a coincidental sequence. The threshold for promoting a parent macro from child invocation patterns needs empirical definition. Bottom-up vs. top-down discovery path selection criteria are also undefined.

### Plan Artifact Capture Quality

When a macro captures a plan artifact (because the repeating subgraph includes the planning-first chain), several gaps remain:

- **Plan staleness** — how to ensure the captured plan remains useful when the macro's execution context diverges from the original. When should the plan be updated vs. preserved as a historical record?
- **Partial plan capture** — what happens when the repeating subgraph includes part of the plan but not all of it? Should the macro capture a partial plan, or only capture plans when the full planning-first chain repeats?
- **Plan vs. hint redundancy** — when both a plan artifact and reasoning hints are captured, there may be overlap in the information they convey. How to avoid redundant context injection to the RPU during escalation?

## Knowledge Extraction Rules (Section 8)

The summarization pipeline describes the flow from situation model to narrative memories to validated knowledge, but the decision criteria for what crosses each threshold are underspecified. The document discusses confidence decay, temporal validity, and revision chains, but the actual extraction rules — what qualifies as knowledge versus preference versus temporary context — require further definition. This will likely emerge from implementation and testing rather than from theoretical design.

### Knowledge Derived from Skills

Knowledge entries extracted from skill execution traces carry provenance links. This introduces additional gaps:

- **Provenance-weighted confidence decay** — when a skill is updated, knowledge derived from the old version enters accelerated decay. The decay rate multiplier (how much faster than normal decay) needs empirical tuning.
- **Cross-version knowledge reconciliation** — when the new skill produces a fact that contradicts the old version's derived knowledge, the reconciliation logic (supersede vs. coexist with validity windows) needs definition.

### Temporal Pattern Extraction

The mechanism for extracting structured temporal patterns from narrative memory chains (Section 8.2.1) is defined conceptually but not algorithmically. How the summarization pipeline identifies consistent timing from irregular event sequences, how schedule expressions are normalized, and how confidence scores are assigned to extracted patterns all require further specification. The feedback loop (acceptance/denial → knowledge adjustment) is defined, but the initial extraction quality determines whether the loop converges or diverges.

## Cold-Start Problem (Mitigated by Skills)

Skills mitigate the cold-start problem by providing competent behavior from day one. However, a residual gap remains:

- **Skill coverage** — skills only cover tasks their authors anticipated. When no skill exists for a task category, the system falls back to general RPU reasoning until it discovers patterns through its own execution. The gap between installed skill coverage and actual user needs is a user decision, not a system problem. But the system should surface this gap to the user: "no skill covers this task category yet."

## Skill Security Audit (Phase 4)

The offline optimization loop includes a Skill Security Audit activity that analyzes external skills for risk patterns. This is advisory — it raises events but does not block execution. Gaps include:

- **Audit coverage** — static analysis of skill instruction files can enumerate tool accesses, detect known risk patterns (prompt injection vectors, dangerous tool references, data exfiltration patterns), and flag behavioral anomalies. The gap is not analysis capability but rule quality and maintenance: which patterns are detectable, how rules evolve as new attack patterns emerge, and how false positives (flagging benign skills) and false negatives (missing malicious skills) are managed.
- **Rule quality and maintenance** — audit rules need to evolve as new attack patterns emerge. Who maintains the rule set? How are rules tested against known-good and known-bad skills?
- **User notification UX** — how are audit findings surfaced to the user? What level of detail is appropriate? How does the user act on findings (uninstall, modify, ignore)?
- **Manual trigger implementation** — the audit should be triggerable when the user views a skill. The UI flow for triggering, displaying results, and acting on findings needs design.

## Skill Version Migration (Phase 5)

When a skill is updated, derived macros and knowledge entries are affected. Gaps include:

- **Auto-migration vs. demotion** — should the system attempt to auto-migrate derived artifacts to the new skill version, or simply demote them and let re-discovery handle it? Auto-migration is more efficient but risks propagating incorrect behavior. Demotion is safer but temporarily loses optimization.
- **Migration hints for skill authors** — can the system generate guidance for skill authors about what changed in execution behavior between versions? This would help authors understand the impact of their changes on derived artifacts.

## Token Budget Model

The scheduler needs a token budget model to enforce reasoning quotas per chain and per time window. This is not about monetary cost — it is about controlling the amount of reasoning the system consumes. The budget model will define:

- **Per-chain token quotas** — when to pause a chain and require user approval to continue
- **Per-time-window token limits** — when to park non-critical chains until the window resets
- **Token-aware scheduling** — estimating token cost when allocating RPU capacity across competing chains

This will be defined during Phase 1 implementation.

## Failure Recovery

The document discusses replayability and provenance but does not specify the mechanics of recovery when things go wrong mid-execution: tool failures, partially completed plans, inconsistent state after external API changes, or corrupted projections. The event-sourced model makes replay possible, but the decision logic for when to replay, when to abort, and when to escalate requires further design.

## Distributed Conflict Resolution

The architecture is designed around a single-node model with edge-cloud separation. Once multiple edge nodes exist, conflict resolution becomes necessary. The "Git, not blockchain" framing works for a single node, but multi-node scenarios will eventually require some form of conflict resolution strategy — CRDTs, vector clocks, or merge policies. This is deferred until Phase 6.

## EEG and Experimental Modalities (Sections 12, 13)

EEG, EMG, and other biometric inputs are included as potential input modalities. They are experimental extensions, not foundational requirements. The architecture would be functionally unchanged if these modalities were removed. They are included to explore how far the concept can be pushed, not because they are necessary for the core thesis.

## Additional Gaps Identified During Documentation Review

### Cross-Store Reference Resolution

The system uses two data stores with cross-store references. Gaps include:

- **Resolution performance** — resolving an event's artifact reference requires a lookup in the artifact store. The performance characteristics of this cross-store resolution (event hash → artifact ID+version → artifact store lookup) need analysis, particularly under high-concurrency scenarios.
- **Consistency during updates** — when an artifact version is demoted, events referencing that version still exist in the event store. The consistency model for stale references needs definition.

### Artifact Store Backup and Recovery

The artifact version store has different backup requirements than the world-state event store. Gaps include:

- **Retention strategy** — artifacts must be kept as long as any reference points to them. The garbage collection strategy for unreferenced artifact versions needs definition.
- **Backup coordination** — the artifact store and event store have independent backup schedules. Recovery from a partial backup (one store restored, the other not) could create inconsistency. The recovery procedure needs definition.
- **Integrity verification** — how to verify that the artifact store's content-addressed storage has not been corrupted, independently of the event store's hash chain.

### Perception Uncertainty Propagation

Perception processing is described as deterministic, but the models it uses (object detection, speech-to-text) are probabilistic. The propagation of perception confidence through situation model → intent → execution is not formally specified. Decision thresholds at each stage need definition, particularly for cases where low-confidence perception triggers high-confidence intent.

### Query Complexity

Cross-graph reference resolution mechanics and query cost analysis for common patterns are not specified. The indexing strategy performance characteristics need analysis, particularly for cross-graph queries that traverse multiple domains.

### Compression Thesis Non-Monotonicity

Knowledge decay, macro demotion, and revision chains mean that structure accumulation is not monotonic. The net effect on "accumulated structure" over time — does it still grow despite decay? — needs formal analysis. The concept of "effective structure" (accumulated minus decayed) should be defined.

---

**Related:** [TECHNICAL_CONCEPT.md](../TECHNICAL_CONCEPT.md#17-known-gaps-and-future-work) for the original specification. [EVALUATION.md](EVALUATION.md) for how gaps affect evaluation design. [DESIGN_DECISIONS.md](DESIGN_DECISIONS.md#why-defer-known-gaps) for rationale behind deferring these gaps.
