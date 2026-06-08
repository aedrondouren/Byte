# Offline Optimization

> Expands on the Offline Optimization section of [TECHNICAL_CONCEPT.md](../TECHNICAL_CONCEPT.md#14-offline-optimization).
> **Audience:** Technical
> **Prerequisites:** TECHNICAL_CONCEPT.md Sections 1–5, 8–9
> **Status:** Complete

The offline optimization loop runs separately from the real-time runtime but shares the same compute infrastructure. The scheduler supports a dynamic repartition mode where compute allocation shifts toward trace mining, macro discovery, knowledge validation, temporal schedule refinement, background optimization, indexing and refinement, and dataset generation for system improvements.

## Dynamic Repartition Mode

The scheduler dynamically shifts compute allocation between runtime execution and offline optimization:

- **Runtime execution** receives guaranteed minimum capacity to maintain responsiveness for safety-critical and interactive chains.
- **Offline optimization** receives all remaining capacity, scaling up when runtime demand is low and scaling down when runtime demand increases.
- **Reserved capacity** ensures that safety-critical responsiveness is maintained even during heavy offline processing.

The runtime executes; the offline system evolves.

## Offline Activities

### Skill Security Audit (Phase 4)

The system analyzes installed skills for known risk patterns. This is an advisory activity — it raises events but does not block skill execution. The audit can be manually triggered by the user when viewing a skill, or run as a background task during offline periods. This serves as the first implementation of background/offline work structure in the system.

Since skills are optional, this activity only runs when skills are installed. If no skills are present, the audit has nothing to analyze and remains idle.

Audit categories include:

- **Prompt injection vectors** — skill instructions that attempt to override system prompts or escape the RPU contract.
- **Dangerous tool access** — skills that request access to tools with destructive capabilities (file deletion, system modification, external API calls with write access).
- **Data exfiltration patterns** — skills that attempt to send structured data to external endpoints not declared in the skill's tool references.
- **Privilege escalation attempts** — skills that attempt to access kernel-level functions or bypass the scheduler.

Audit findings are recorded as events in the execution graph with category, severity, and affected skill metadata. The user is notified through the active channel. The audit is heuristic-based and will produce false positives and false negatives; the user decides whether to act on findings.

### Trace Mining and Macro Discovery (Phase 5)

Reads from world-state event store, writes to artifact version store. The offline loop scans execution traces for repeated patterns using sliding window mining and subgraph matching. This is computationally expensive and runs only during low-demand periods.

### Skill-to-Macro Transformation (Phase 5)

Reads from world-state event store, writes to artifact version store. A specialized discovery path analyzes skill execution traces for repeated patterns. Unlike general trace mining, skill-derived discovery has access to the skill's instruction file as an intent reference, making the equivalence check more tractable. The transformation pipeline:

1. Identifies repeated subgraphs within skill execution traces.
2. Extracts parameters from the skill's configurable values and observed user overrides.
3. Distills reasoning at branch points into Reasoning Hints, validated against the skill's stated intent.
4. Proposes a skill-derived macro with provenance metadata linking it to the source skill and version.
5. Validates the macro against historical skill execution traces.

### Hierarchical Macro Discovery (Phase 5)

Reads from artifact version store, writes to artifact version store. The offline loop identifies composition patterns among existing macros. When child macros are invoked together repeatedly in a consistent sequence, the system proposes a parent macro that composes them. This runs as a separate pass after leaf macro discovery, operating at the macro invocation level rather than the tool call level.

### Knowledge Validation (Phase 4)

Reads from world-state event store, writes to world-state event store. Proposed facts from the summarization pipeline are tested against accumulated evidence. Contradiction detection, consistency checks, and confidence scoring run as batch processes.

### Skill Version Impact Analysis (Phase 5)

Reads from both stores, writes to both stores. When a skill is updated, the offline loop re-validates all derived artifacts:

1. Identifies all macros with provenance to the old skill version.
2. Replays them against execution traces from the new skill version.
3. Proposes new macro versions with updated provenance if behavior is equivalent.
4. Identifies all knowledge entries with provenance to the old skill version.
5. Accelerates confidence decay for entries not corroborated by new skill executions.

### Temporal Schedule Refinement (Phase 4)

Reads from world-state event store, writes to world-state event store. Recurring schedules are adjusted based on observed behavior — shifting timing to better match actual usage patterns, merging overlapping schedules, and pruning schedules whose underlying knowledge has decayed below the activation threshold.

### Background Indexing and Refinement (Phase 4)

Reads from and writes to world-state event store. Memory and knowledge graph indexes are rebuilt with improved algorithms. Semantic similarity indexes are updated as new experiences accumulate.

### Dataset Generation (Phase 4)

Reads from world-state event store. Historical perception, situation model, and execution traces are packaged as training datasets for improving perception models, situation model generation, and macro discovery algorithms.

### Entity State Refinement (Phase 4)

Reads from world-state event store, writes to entity registry. The offline loop updates entity state projections based on accumulated events:

1. Scans recent events for entity participation (subjects, participants, identifiers).
2. Updates entity state (last_seen, interaction_count, relationship strength).
3. Checks for potential identity matches across channels (same name, voice print, behavioral patterns).
4. Generates merge suggestions for Admin or delegated review.

### Permission-Impact Analysis (Phase 5)

Reads from both stores, writes to both stores. When entity permissions change, the offline loop identifies and invalidates affected derived artifacts:

1. Identifies all knowledge entries where `learned_from` == changed entity.
2. Applies accelerated confidence decay to affected knowledge entries.
3. Identifies all macros derived from changed entity's execution.
4. Demotes affected macros (status: `source_permission_changed`).
5. If entity is re-elevated, flags affected artifacts for re-validation.

## Reserved Capacity Guarantees

Even during heavy offline processing:

- **Critical chains** (safety, security, system integrity) receive non-interruptible capacity.
- **Interactive chains** (user-facing responses) receive guaranteed minimum latency.
- **The offline loop is interruptible.** If runtime demand spikes, offline activities are paused and resumed when capacity is available.

---

**Related:** [TECHNICAL_CONCEPT.md](../TECHNICAL_CONCEPT.md#14-offline-optimization) for the original specification. [EVALUATION.md](EVALUATION.md) for how offline optimization affects evaluation metrics.
