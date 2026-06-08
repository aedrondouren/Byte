# Architecture Flow Diagrams

> Expands on the Architecture Flow Diagrams section of [TECHNICAL_CONCEPT.md](../TECHNICAL_CONCEPT.md#18-architecture-flow-diagrams).
> **Audience:** Technical
> **Prerequisites:** TECHNICAL_CONCEPT.md Sections 1–18
> **Status:** Complete

## Full Transformation Flow

```
Skills (optional, pre-authored, versioned, executed via RPU)
    ↓
Raw Signals → Perception Processing → Perception
    ↓
Situation Model Generation → Situation Model
    ↓
Intent Estimation → Intent
    ↓
Projection → Execution Graph
    ↓                                    ↗
Projection → Memory Graph               │ (skill execution traces, if skills installed)
    ↓                                    │
Projection → Knowledge Graph ←──────────┘
    │                    ↗
    ├── Temporal Patterns → Temporal Intent → Intent (feedback loop)
    │
    └── Channel → External Surface (Web, Discord, CLI, API, etc.)

┌──────────────────────────────────────────────────────────┐
│  World-State Event Store (append-only, content-addressed) │
│  Perception Graph → Execution Graph → Memory Graph       │
│  → Knowledge Graph                                        │
└──────────────────────┬───────────────────────────────────┘
                       │ content hash → artifact ID+version
┌──────────────────────▼───────────────────────────────────┐
│  Artifact Version Store (content-addressed, versioned)    │
│  Macro Graph (skill-derived + pattern-derived)            │
│  Skill Registry                                           │
│  Code Registry                                            │
└──────────────────────────────────────────────────────────┘
```

## Two-Store Architecture

```
┌─────────────────────────────────────────┐
│         World-State Event Store          │
│  Append-only, content-addressed events   │
│  State = projection of event history     │
│                                          │
│  Perception Graph  ── what happened      │
│  Execution Graph   ── what was done      │
│  Memory Graph      ── what was felt      │
│  Knowledge Graph   ── what is known      │
│                                          │
│  Queries: temporal, causal, semantic     │
│  Retention: time-window based, archival  │
└──────────────────┬──────────────────────┘
                   │ content hash → artifact ID+version
┌──────────────────▼──────────────────────┐
│         Artifact Version Store           │
│  Content-addressed, semantically         │
│  versioned, lifecycle-managed            │
│  State = latest version per artifact     │
│                                          │
│  Macro Graph     ── compiled patterns    │
│  Skill Registry  ── authored behaviors   │
│  Code Registry   ── tested components    │
│                                          │
│  Queries: identity, version, capability  │
│  Retention: reference-based, no archival │
└─────────────────────────────────────────┘
```

## Unified System Model

```
World-State Event Store  ◄──►  Artifact Version Store
    │                               │
    └───────────┬───────────────────┘
                │
        Cognitive Kernel
                │
        ┌───────┴───────┐
        │               │
      RPU         Semantic Orchestration
    (reasoning)    (Tool / App Services)
        │               │
        └───────┬───────┘
                │
        Macro Optimization
    (discovery, validation, audit)
                │
           Channels
    (Web, Discord, CLI, API)
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
Sources:
  ├── Skill Execution Traces (provenance-linked) [when skills installed]
  └── General Execution Traces (pattern-discovered) [always]
    ↓
Pattern Discovery (sliding window + subgraph matching)
    ↓
Macro Proposal (LLM-assisted normalization)
    ↓
Validation (replay + equivalence + edge cases + provenance check + plan check if captured)
    ↓
Compilation (deterministic execution unit, captures whatever repeated)
    ↓
Execution (with Reasoning Hints, captured plan artifact if present)
    ↓
Demotion (if usage decays, primitives change, source skill updates, or child demoted)
```

## Macro Captured Content Spectrum

A macro captures whatever the discovery pipeline detects as the repeating subgraph:

```
Level 1: Tool Calls Only
    [fetch_weather] → [format_response] → [send_to_channel]
    (no branching, no plan artifact)

Level 2: Tool Calls + Reasoning Hints
    If condition_met:
        Hint: "User prefers detailed forecast"
        [fetch_weather_detailed]
    Else:
        Hint: "User prefers brief summary"
        [fetch_weather_brief]
    [format_response] → [send_to_channel]
    (branching with hints, no plan artifact)

Level 3: Tool Calls + Reasoning Hints + Plan Artifact
    Captured Plan: "Morning Briefing: weather → calendar → news"
    If condition_met:
        Hint: "User prefers detailed forecast"
        [fetch_weather_detailed]
    Else:
        Hint: "User prefers brief summary"
        [fetch_weather_brief]
    [fetch_calendar] → [fetch_news] → [format_response] → [send_to_channel]
    (full planning-first chain repeated, all content captured)
```

The macro does not mandate what is captured. It compiles the detected pattern as-is.

## Skill → Macro → Knowledge Provenance Chain

```
Skill v1.0 (installed by user)
    ↓ executed 47 times via RPU
    ↓
Macro morning_briefing_v3
    provenance: { source: "skill_morning_briefing", version: "v1.0" }
    ↓
Knowledge: "user prefers 7AM briefing"
    provenance: { source: "skill_morning_briefing", version: "v1.0" }
    ↓
Skill updated to v1.1
    ↓
Macro flagged for re-validation → demoted (status: source_updated)
    → falls back to skill execution via RPU
Knowledge confidence decays (accelerated)
    → re-corroborated by v1.1 executions → new knowledge entry with v1.1 provenance
    ↓
New macro proposed from v1.1 traces → validated → promoted
```

## Hierarchical Macro Composition

```
Macro: morning_routine (parent)
    ├── Macro: weather_check (child subgraph reference)
    │       └── Tool: fetch_weather
    │       └── Tool: format_response
    ├── Macro: calendar_briefing (child subgraph reference)
    │       └── Tool: fetch_calendar
    │       └── Tool: format_summary
    └── Macro: news_summary (child subgraph reference)
            └── Tool: fetch_news
            └── Tool: rank_by_interest

At execution time: child macros are inlined into parent's sequence.
No runtime call stack — just sequential/parallel event execution.
Always expandable: "expand all" reveals full tool-call chain.
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
Edge Node                          Homelab
┌──────────────────┐               ┌──────────────────────────┐
│ Perception       │  structured   │  Stores + Kernel + RPU   │
│ Processing       │──events──────▶│  + Orchestration         │
│ Local Inference  │◀─requests─────│  + Optimization          │
│ Immediate Response│               │  + Channels              │
└──────────────────┘               └──────────┬───────────────┘
                                              │ normalized
                                              │ requests
                                              ▼
                                       ┌──────────────┐
                                       │  AI Models   │
                                       │ (LLM Runtime)│
                                       └──────────────┘
```

---

**Related:** [GRAPH.md](GRAPH.md) for world-state graph diagrams. [TECHNICAL_CONCEPT.md](../TECHNICAL_CONCEPT.md#18-architecture-flow-diagrams) for the original specification.
