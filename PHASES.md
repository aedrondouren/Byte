# Build Phases

> Canonical source for B.Y.T.E. build phase status, dependencies, and document mapping.
> **Audience:** Everyone
> **Status:** Active

---

## Phase Overview

Phases 1–4 test the core thesis. Phase 4 (Memory + Knowledge) is the falsification point. Phases 5–6 are experimental extensions built on that foundation.

| Phase | Component                           | Status          | Core Documents                                                                                                       |
| ----- | ----------------------------------- | --------------- | -------------------------------------------------------------------------------------------------------------------- |
| 1     | Kernel + Execution Graph            | Design complete | [GRAPH.md](docs/core/GRAPH.md)                                                                                       |
| 2     | RPU + Orchestration                 | Design complete | [RPU.md](docs/core/RPU.md), [ORCHESTRATION.md](docs/core/ORCHESTRATION.md)                                           |
| 3     | Signal-to-Intent Pipeline           | Design complete | —                                                                                                                    |
| 4     | Memory + Knowledge + Skill Registry | Design complete | [MACROS.md](docs/core/MACROS.md) (skill execution)                                                                   |
| 5     | Macros + Code Registry              | Research phase  | [MACROS.md](docs/core/MACROS.md), [REGISTRY.md](docs/core/REGISTRY.md)                                               |
| 6     | Edge + Multimodal                   | Conceptual      | [EDGE_ARCHITECTURE.md](docs/core/EDGE_ARCHITECTURE.md), [MULTIMODAL_INTERFACE.md](docs/core/MULTIMODAL_INTERFACE.md) |

---

## Phase Dependencies

```
Phase 1 (Kernel + Execution Graph)
    ↓
Phase 2 (RPU + Orchestration)
    ↓
Phase 3 (Signal-to-Intent Pipeline)
    ↓
Phase 4 (Memory + Knowledge + Skill Registry)  ← FALSIFICATION POINT
    ↓
Phase 5 (Macros + Code Registry)               ← EXPERIMENTAL
    ↓
Phase 6 (Edge + Multimodal)                    ← EXPERIMENTAL
```

Phases 1–4 are sequential and required for the core thesis test. Phases 5–6 depend on Phase 4 but are not required for the thesis.

---

## Phase Descriptions

### Phase 1: Kernel + Execution Graph

The irreplaceable core. Manages execution chains as DAG-based computation graphs, handles scheduling, prioritization, interruption, and resource arbitration. Records all activity as immutable events in the world-state graph.

**Sections:** [TECHNICAL_CONCEPT.md Section 4–5](docs/TECHNICAL_CONCEPT.md#4-world-state-graph-architecture)

### Phase 2: RPU + Orchestration

The reasoning coprocessor and semantic orchestration layer. The RPU receives structured requests and returns structured results. Orchestration primitives provide deterministic tool composition.

**Sections:** [TECHNICAL_CONCEPT.md Section 6, 10](docs/TECHNICAL_CONCEPT.md#6-reasoning-processing-unit)

### Phase 3: Signal-to-Intent Pipeline

Converts raw signals into actionable intent through three layers: perception processing, situation model generation, and intent estimation.

**Sections:** [TECHNICAL_CONCEPT.md Section 7](docs/TECHNICAL_CONCEPT.md#7-signal-to-intent-pipeline)

### Phase 4: Memory + Knowledge + Skill Registry

The summarization pipeline compresses experience into narrative memories and validated knowledge. Skills provide optional seed behaviors. **This is the falsification point** — if Phase 4 does not demonstrate a measurable reduction in reasoning cost for equivalent outcomes compared to the Phase 2 baseline, the core thesis fails.

**Sections:** [TECHNICAL_CONCEPT.md Section 8–9.3](docs/TECHNICAL_CONCEPT.md#8-event-summarization-and-indexing)
**Evaluation:** [EVALUATION.md](docs/core/EVALUATION.md)

### Phase 5: Macros + Code Registry (Experimental)

The optimization layer. Discovers repeated execution patterns and compiles them into deterministic macros. The code registry provides tested, versioned components for software development.

**Sections:** [TECHNICAL_CONCEPT.md Section 9.1–9.2](docs/TECHNICAL_CONCEPT.md#91-macro-system)

### Phase 6: Edge + Multimodal (Experimental)

Portable Personal Edge Node for mobile perception and the multimodal cognitive interface for sensor fusion. The core thesis does not depend on these existing.

**Sections:** [TECHNICAL_CONCEPT.md Section 12–13](docs/TECHNICAL_CONCEPT.md#12-edge-architecture)

---

## Thesis Test

The core thesis — "can accumulated structure substitute for repeated inference?" — is tested by comparing Phase 4 against the Phase 2 baseline:

- **Baseline (Phase 2):** Structured but not compressed. Planning-first workflows with structured contracts but no memory, knowledge, or macros.
- **Test (Phase 4):** Memory + Knowledge graphs active. Validated facts eliminate re-derivation. Narrative memories replace raw context.
- **Criterion:** Statistically significant reduction (p < 0.05) in reasoning cost for equivalent task outcomes.

Skills are available from Phase 4 as a harness completeness feature but are not part of the thesis test — they are a constant across both baseline and test, and therefore cancel out.

---

**Related:** [README.md](README.md) for project overview. [TECHNICAL_CONCEPT.md](docs/TECHNICAL_CONCEPT.md#32-build-phases-and-document-mapping) for section-to-phase mapping. [EVALUATION.md](docs/core/EVALUATION.md) for evaluation methodology.
