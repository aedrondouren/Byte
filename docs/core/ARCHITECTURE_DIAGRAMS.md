# Architecture Flow Diagrams

> Expands on the Architecture Flow Diagrams section of [TECHNICAL_CONCEPT.md](../TECHNICAL_CONCEPT.md#18-architecture-flow-diagrams).
> **Audience:** Technical
> **Prerequisites:** TECHNICAL_CONCEPT.md Sections 1–18
> **Status:** Complete

This document contains the simple flow diagrams used across the architecture. Complex visual diagrams have been replaced with prose descriptions and references to authoritative locations.

---

## System Architecture (Prose)

The system has two stores connected to a cognitive kernel:

- **World-State Event Store** — append-only, content-addressed events. Contains Perception Graph (what happened), Execution Graph (what was done). State is derived by projecting event history. Queries are temporal, causal, and semantic. Retention is time-window based with archival to cold storage.
- **Artifact Version Store** — content-addressed, semantically versioned, lifecycle-managed. Contains Memory Graph (experienced events, versioned), Knowledge Graph (validated facts, versioned), Macro Graph (compiled patterns), Skill Registry (authored behaviors), Code Registry (tested components), Entity Graph (identity and permissions). State is the latest version per artifact. Queries are identity, version, capability, and semantic based. Retention is reference-based — artifacts are kept as long as anything points to them.

Cross-store references use content hashes in events pointing to artifact ID+version. No unified index is needed.

For the full two-store specification, see [TECHNICAL_CONCEPT.md Section 1.6](../TECHNICAL_CONCEPT.md#16-implementation-note-two-stores-eight-graphs). For query complexity analysis, see [GRAPH.md](GRAPH.md#query-complexity-and-cross-graph-resolution).

---

## Entity Graph Position (Prose)

The Entity Graph is the sixth artifact graph in the Artifact Version Store, alongside Memory, Knowledge, Macro, Skill, and Code Registry graphs. It stores versioned entity definitions — identity, trust level, permissions, and relationships — as content-addressed, lifecycle-managed artifacts. Entity state (last seen, interaction count, derived preferences) is a projection over the world-state event stream, not stored as an artifact.

The Cognitive Kernel reads from both stores, invokes the RPU for reasoning, and executes through semantic orchestration primitives.

For the full entity model, see [ENTITIES.md](ENTITIES.md).

---

## Entity Definition Lifecycle (DAG)

```
Unknown Identifier (raw channel ID, no Entity Graph entry)
    ↓
[Entity Def v1: discovered from channel_A]          [Entity Def v1: discovered from channel_B]
  type: external                                      type: external
  trust: untrusted                                    trust: untrusted
  identity: {discord:@bob}                            identity: {voice_print_xyz}
                            \                        /
                             \                      /
                    [Entity Def v2: merged identity]
                    ├── identity: { primary: discord:@bob, merged: [voice_print_xyz] }
                    ├── trust: untrusted
                    └── provenance: { merged_from: [v1_A, v1_B] }
                              |
                    [Entity Def v3: permissions elevated]
                    ├── trust: trusted_user
                    ├── permissions: { domain_access: { technical: read } }
                    └── provenance: { elevated_by: admin, from_version: v2 }
                              |
                    [Entity Def v4: relationships updated]
                    ├── trust: trusted_user
                    ├── relationships: [ { target: entity_admin, type: collaborates_with } ]
                    └── provenance: { updated_by: system, from_version: v3 }
                              |
                    [Entity Def v5: demoted]
                    ├── trust: untrusted
                    ├── permissions: { none }
                    ├── identity: { preserved }           ← system still knows who this is
                    ├── relationships: [ preserved ]      ← relationship history persists
                    └── provenance: { demoted_by: admin, from_version: v4 }
```

---

## Entity/Session Composition (Prose)

At runtime, the complete entity context is composed from three sources:

```
Entity Definition (Entity Graph, artifact store)
    +
Entity State (projection from event stream)
    +
Channel Binding (entity_bindings in Channel definition)
    =
Complete Entity/Session Context (runtime)
```

The Entity Definition provides identity, trust level, permissions, and relationships. Entity State provides last seen, interaction count, and derived preferences. Channel Binding provides the session context — which channel, session ID, and role. Effective permissions are the intersection of entity permissions and channel permissions.

For the full composition model with examples, see [ENTITIES.md Entity/Session Composition](ENTITIES.md#entitysession-composition).

---

## Entity Demotion Preserves Knowledge (Prose)

When an entity is demoted:

- **Entity Definition** — trust level and permissions are reduced, but identity and relationships are preserved. The system retains understanding of who this entity is.
- **Entity State** — unchanged. All historical data persists (last seen, interaction count, preferences).
- **World-State Graphs** — unchanged. Memories, knowledge entries, and relationships persist.

Result: the system knows WHO this entity is, but grants NO runtime access. If re-elevated, all context is immediately available.

For the full demotion model with before/after examples, see [ENTITIES.md Demotion and Session Composition](ENTITIES.md#demotion-and-session-composition).

---

## Unified System Model (Prose)

The system layers from bottom to top:

```
World-State Event Store ◄──► Artifact Version Store
    │
    └── Cognitive Kernel
         ├── RPU (reasoning)
         ├── Semantic Orchestration (Tool / App Services)
         ├── Macro Optimization (discovery, validation, audit)
         └── Channels (Web, Discord, CLI, API)
```

The stores hold all system state. The kernel governs execution. The RPU provides structured reasoning. Semantic orchestration provides deterministic tool composition. The optimization layer compresses experience into reusable structure. Channels expose projected state to external surfaces.

Edge nodes run perception processing and local inference, sending only structured events to the homelab. AI models are accessed through the LLM Runtime adapter and are not part of the core architecture.

---

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

---

## Summarization Pipeline

```
Situation Model (event store, high volume, detailed)
    ↓
[Memory Extraction Projection]
    ↓
Memory Artifacts (artifact store, compressed, contextual, versioned)
    ↓
[Knowledge Generalization Projection]
    ↓
Knowledge Artifacts (artifact store, facts, patterns, schedules, versioned)
```

---

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

---

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

---

## Skill → Macro → Knowledge Provenance Chain

```
Skill v1.0 (installed by user, artifact store)
    ↓ executed 47 times via RPU
    ↓
Execution Traces (event store)
    ↓
Macro morning_briefing_v3 (artifact store)
    provenance: { source: "skill_morning_briefing", version: "v1.0" }
    ↓
Memory Artifacts (artifact store)
    ↓
Knowledge: "user prefers 7AM briefing" (artifact store)
    provenance: { source: "skill_morning_briefing", version: "v1.0" }
    ↓
Skill updated to v1.1
    ↓
Macro flagged for re-validation → demoted (status: source_updated)
    → falls back to skill execution via RPU
Knowledge confidence decays (accelerated)
    → re-corroborated by v1.1 executions → new knowledge artifact with v1.1 provenance
    ↓
New macro proposed from v1.1 traces → validated → promoted
```

---

## Hierarchical Macro Composition

```
Macro: morning_routine (parent)
├── Macro: weather_check (child subgraph reference)
│   └── Tool: fetch_weather
│   └── Tool: format_response
├── Macro: calendar_briefing (child subgraph reference)
│   └── Tool: fetch_calendar
│   └── Tool: format_summary
└── Macro: news_summary (child subgraph reference)
    └── Tool: fetch_news
    └── Tool: rank_by_interest

At execution time: child macros are inlined into parent's sequence.
No runtime call stack — just sequential/parallel event execution.
Always expandable: "expand all" reveals full tool-call chain.
```

---

## Cognitive Cache Hierarchy

```
L1 — Active Context      (fastest, most expensive to maintain)
L2 — Structured State
L3 — World Model
L4 — Event History
L5 — Reasoning (RPU)     (slowest, most expensive to invoke)
```

---

## Edge-Cloud Separation (Prose)

- **Edge Node:** Runs Perception Processing, Local Inference, and Immediate Response. Sends structured events to the homelab. Receives requests back.
- **Homelab:** Runs Stores, Kernel, RPU, Orchestration, Optimization, and Channels. Receives structured events from edge nodes. Sends normalized requests to AI Models.
- **AI Models (LLM Runtime):** Receives normalized requests only. Not part of the core architecture.

For the full edge specification, see [EDGE_ARCHITECTURE.md](EDGE_ARCHITECTURE.md). For the edge-cloud separation in TECHNICAL_CONCEPT.md, see [Section 12 Edge Architecture](../TECHNICAL_CONCEPT.md#12-edge-architecture).

---

**Related:** [GRAPH.md](GRAPH.md) for world-state graph diagrams. [TECHNICAL_CONCEPT.md](../TECHNICAL_CONCEPT.md#18-architecture-flow-diagrams) for the original specification.
