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

### Trace Mining and Macro Discovery

The offline loop scans execution traces for repeated patterns using sliding window mining and subgraph matching. This is computationally expensive and runs only during low-demand periods.

### Knowledge Validation

Proposed facts from the summarization pipeline are tested against accumulated evidence. Contradiction detection, consistency checks, and confidence scoring run as batch processes.

### Temporal Schedule Refinement

Recurring schedules are adjusted based on observed behavior — shifting timing to better match actual usage patterns, merging overlapping schedules, and pruning schedules whose underlying knowledge has decayed below the activation threshold.

### Background Indexing and Refinement

Memory and knowledge graph indexes are rebuilt with improved algorithms. Semantic similarity indexes are updated as new experiences accumulate.

### Dataset Generation

Historical perception, situation model, and execution traces are packaged as training datasets for improving perception models, situation model generation, and macro discovery algorithms.

## Reserved Capacity Guarantees

Even during heavy offline processing:

- **Critical chains** (safety, security, system integrity) receive non-interruptible capacity.
- **Interactive chains** (user-facing responses) receive guaranteed minimum latency.
- **The offline loop is interruptible.** If runtime demand spikes, offline activities are paused and resumed when capacity is available.

---

**Related:** [TECHNICAL_CONCEPT.md](../TECHNICAL_CONCEPT.md#14-offline-optimization) for the original specification. [EVALUATION.md](EVALUATION.md) for how offline optimization affects evaluation metrics.
