# Architecture Flow Diagrams

> Expands on the Architecture Flow Diagrams section of [TECHNICAL_CONCEPT.md](../TECHNICAL_CONCEPT.md#18-architecture-flow-diagrams).
> **Audience:** Technical
> **Prerequisites:** TECHNICAL_CONCEPT.md Sections 1–11
> **Status:** Complete

## Full Transformation Flow

```
Raw Signals
    ↓
Perception Processing → Perception
    ↓
Situation Model Generation → Situation Model
    ↓
Intent Estimation → Intent
    ↓
Projection → Execution Graph
    ↓
Projection → Memory Graph
    ↓
Projection → Knowledge Graph
    │
    ├── Temporal Patterns → Temporal Intent → Intent (feedback loop)
    │
    └── Channel → External Surface (Web, Discord, CLI, API, etc.)
```

## Unified System Model

```
World-State Graph (canonical event IR)
    │
    ├── Cognitive Kernel
    ├── Perception Nodes
    └── Interaction Layer
            │
            ▼
    Execution + Trace IR Layer
            │
            ▼
    Macro Optimization (reversible JIT)
            │
            ▼
    Language Adapters (TS / Go / C++ etc.)
```

## Intent Pipeline (Three-Stage Flow)

```
Intent (what is desired)
    ↓
Proposal (how to do it)
    ↓
Commit (authorized to execute)
    ↓
Execution Chain
```

## Summarization Pipeline

```
Situation Model (high volume, detailed)
    ↓
Narrative Memories (compressed, contextual)
    ↓
Validated Knowledge (facts, patterns, schedules)
```

## Macro Lifecycle

```
Execution Traces
    ↓
Pattern Discovery (sliding window + subgraph matching)
    ↓
Macro Proposal (LLM-assisted normalization)
    ↓
Validation (replay + equivalence + edge cases)
    ↓
Compilation (deterministic execution unit)
    ↓
Execution (with Reasoning Hints)
    ↓
Demotion (if usage decays or primitives change)
```

## Cognitive Cache Hierarchy

```
L1 — Active Context      (fastest, most expensive to maintain)
L2 — Structured State
L3 — World Model
L4 — Event History
L5 — Reasoning (RPU)     (slowest, most expensive to invoke)
```

## Edge-Cloud Separation

```
Edge Device                          Homelab
┌──────────────────┐                 ┌──────────────────┐
│ Perception       │  structured     │ Heavy reasoning  │
│ Processing       │────events──────▶│ Long-term memory │
│ Local Inference  │◀────requests────│ Planning         │
│ Immediate Response│                 │ Knowledge valid. │
└──────────────────┘                 └──────────────────┘
```

---

**Related:** [GRAPH.md](GRAPH.md) for world-state graph diagrams. [TECHNICAL_CONCEPT.md](../TECHNICAL_CONCEPT.md#18-architecture-flow-diagrams) for the original specification.
