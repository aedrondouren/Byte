# Distributed Cognitive Runtime

**Technical Concept Document**

---

## Abstract

This document describes B.Y.T.E. (Behavior Yielding Through Evolution), a distributed, event-sourced cognitive runtime architecture that unifies perception, reasoning, and action through a Git-like world-state graph. The system tests a single hypothesis: **can accumulated structure substitute for repeated inference?** Instead of improving AI agents through larger models or more context, the architecture converts past reasoning into reusable structure — macros and knowledge graphs — so that the minimum reasoning required to achieve a task decreases over time. A companion code registry provides tested, versioned components for software development tasks. The core thesis is falsifiable: if Phase 4 (Memory + Knowledge) does not demonstrate a measurable reduction in reasoning cost for equivalent outcomes, the project fails. This document defines the complete target architecture. Implementation has not yet begun.

---

## Related Documents

### Core Subsystems

- [GRAPH.md](core/GRAPH.md) — World-state graph model (diagrams, design heuristics, query complexity, trust hierarchy)
- [RPU.md](core/RPU.md) — Reasoning Processing Unit (contract, error handling, model versioning)
- [ORCHESTRATION.md](core/ORCHESTRATION.md) — Semantic orchestration primitives (semantics, composition laws)
- [MACROS.md](core/MACROS.md) — Macro system (mining algorithms, validation, demotion, skill-derived macros, hierarchical composition)
- [REGISTRY.md](core/REGISTRY.md) — Code registry (versioning, dependency resolution, transpilation)
- [ENTITIES.md](core/ENTITIES.md) — Entity model (schema, trust levels, lifecycle, permissions)
- [CHANNELS.md](core/CHANNELS.md) — Channel architecture (approval, bidirectional flow, control signals)
- [RETRIEVAL.md](core/RETRIEVAL.md) — Retrieval pipeline (filter chain, dual-access projection)

### Experimental Extensions

- [EDGE_ARCHITECTURE.md](core/EDGE_ARCHITECTURE.md) — Portable Personal Edge Node (hardware, power, networking)
- [MULTIMODAL_INTERFACE.md](core/MULTIMODAL_INTERFACE.md) — Multimodal cognitive interface (sensor fusion, thresholds)
- [OFFLINE_OPTIMIZATION.md](core/OFFLINE_OPTIMIZATION.md) — Background optimization loop

### Research & Evaluation

- [EVALUATION.md](core/EVALUATION.md) — Evaluation methodology (baselines, ablation, power analysis)
- [KNOWN_GAPS.md](core/KNOWN_GAPS.md) — Open problems and future work
- [DESIGN_DECISIONS.md](core/DESIGN_DECISIONS.md) — Rationale behind key architectural choices
- [THREAT_MODEL.md](core/THREAT_MODEL.md) — Attacker capability analysis and threat scenarios

### Reference

- [REFERENCE.md](REFERENCE.md) — Glossary and terminology
- [ARCHITECTURE_DIAGRAMS.md](core/ARCHITECTURE_DIAGRAMS.md) — All system flow diagrams
- [DOCUMENT_MAP.md](DOCUMENT_MAP.md) — Document dependency graph and reading order

---

## 1. Introduction

### 1.1 Problem Statement

Current AI agent architectures share a common flaw: they embed memory, planning, identity, world state, execution management, and communication into a single reasoning loop — typically a large language model prompt. This makes agents expensive (every interaction requires full inference), fragile (no persistent state between sessions), and opaque (no audit trail of decisions). Agent frameworks like LangChain and AutoGen orchestrate prompt chains but do not address the fundamental issue: **repeated inference for repeated behavior**.

### 1.2 What Byte Replaces

| Before                | Byte                                                                            |
| --------------------- | ------------------------------------------------------------------------------- |
| Agent loops           | Execution chains — persistent, prioritized, interruptible                       |
| Prompt chains         | Planning-first workflows — plan, execute, observe, compress                     |
| Chatbots              | Channels over world-state — state is primary, conversation is output            |
| External dependencies | Code registry — tested, versioned, in-house components for software development |

### 1.3 Core Thesis

This project tests a single hypothesis: **can accumulated structure substitute for repeated inference?**

Instead of making agents smarter through larger models, more context, or additional reasoning passes, Byte asks what is the minimum reasoning required to achieve a given task in a given domain, and whether that minimum decreases over time as the system converts past reasoning into reusable structure.

The mechanism is a continuous loop managed by the kernel:

```
(Optional: Skills) → Reasoning → Execution → History → Compression → Structure → Less future reasoning
```

Skills are an optional acceleration — the system works without them, discovering patterns organically from its own execution. When skills are installed, they provide competent behavior from the start; as they execute, their traces enter the event graph and the compression pipeline converts skill-driven reasoning into derived macros and knowledge. Without skills, the same compression pipeline operates on general execution traces, discovering patterns from the system's own organic behavior.

Every layer in this document exists to support that loop. The event graph stores it. The scheduler executes it. The memory system compresses it. The macro system formalizes it. The skill registry optionally seeds it.

**Non-monotonicity of structure accumulation.** The compression thesis assumes that accumulated structure grows over time, but the system includes mechanisms that reduce structure: knowledge decay (Section 8.2), macro demotion (Section 9.1.4), and revision chains (Section 8.2). This means structure accumulation is not monotonic — some structure is lost as it becomes obsolete or invalid.

The net effect is captured by the concept of **effective structure**: the total accumulated structure minus decayed, demoted, or superseded structure. The thesis claims that effective structure grows over time, even though gross structure (total ever accumulated) may exceed net structure (currently valid). The evaluation methodology (Section 16) measures reasoning cost reduction, which implicitly measures effective structure growth — if reasoning cost decreases, effective structure is growing.

The decay mechanisms are intentional: they prevent the system from accumulating stale or incorrect structure. A system that never forgets becomes burdened by outdated knowledge. The decay rates (technical facts: slow, social facts: faster, environmental facts: fastest) are designed to match the expected volatility of each domain.

### 1.4 Core Invariants

These principles govern every design decision:

- **AI proposes intent; the kernel executes deterministically.** The LLM never runs the system. It proposes structured intent into the execution kernel, which validates, schedules, and executes it using deterministic primitives.
- **Everything is an event.** There is no distinction between input and output. Sensor readings, tool calls, reasoning steps, execution chains, and environmental state are all events in the same structure.
- **The world-state is an immutable event graph, not a mutable database.** Events are append-only, content-addressed, and cryptographically verifiable. Current state is always a projection of history.
- **Execution chains are persistent, prioritized, and interruptible.** Chains survive waiting, suspension, and resumption. They are governed by priority levels and resource quotas.
- **Macros are compiled execution subgraphs, not behavior overrides.** Repeated execution patterns are discovered, validated, and compiled into reusable units. They never replace primitives. Macros may be derived from skill execution traces (carrying provenance links) or discovered from general execution patterns. Macros may compose child macros hierarchically as DAG subgraph references.
- **Skills are optional seed behaviors; derived artifacts carry provenance.** Macros and knowledge extracted from skill execution traces maintain causal links to their source skill and version. When a skill is updated, derived artifacts are flagged for re-validation through the existing demotion mechanics. Skills use the existing harness format — users install skills they trust; the system optimizes them automatically. The system operates without any skills installed, accumulating structure from its own organic execution.
- **Edge-cloud separation of cognition.** Real-time perception and immediate interaction logic run on edge devices. Heavy reasoning, long-term memory, and planning run on the homelab.
- **AI is an interpretation layer, not a control system.** Infrastructure and deterministic logic never depend on AI reasoning.
- **Reasoning is externalized into accumulated structure.** The objective is not to maximize model intelligence but to minimize the amount of reasoning required to produce intelligent behavior.
- **Temporal intent is kernel-scheduled, not AI-proposed.** The temporal intent generator operates as a deterministic cron subsystem. The RPU identifies temporal patterns and presents scheduled or one-time job suggestions to the user. Upon approval, these become cron entries that fire automatically. Each tool service declares its own destructiveness level (read-only vs. state-changing); non-destructive entries may auto-execute once confidence crosses a configurable threshold, while destructive entries always require per-execution approval. User dismissal feeds back into confidence decay. This does not violate the "AI proposes intent; the kernel executes" invariant — the AI proposes the schedule, the kernel executes the schedule.
- **Graceful degradation is mandatory.** The system must remain functional and safe when disconnected from the homelab, when network links fail, or when sensor inputs become unreliable.
- **Admin is the root of trust.** A single Admin entity exists from bootstrap, linked to the primary webUI. All other entities start untrusted. Trust is never automatic — it is explicitly granted by a higher-trust entity.
- **Admin ceiling is irrevocable.** No permission delegation chain can produce effective permissions equal to or greater than Admin. Admin is always the ceiling.
- **Memory is not owned by entities.** All memory lives in the shared graph with metadata describing context. Entities are retrieval policy definitions, not memory containers.
- **History is immutable; derivatives are invalidated.** Memories and events are never modified. Knowledge and macros are invalidated when source entity permissions change.
- **Privacy is structural.** Inferred from who is involved and what the memory concerns, encoded as metadata that survives abstraction into knowledge.
- **Knowledge propagates; context remains scoped.** Factual knowledge can become globally accessible while relationship and provenance context stays bound to the originating entity relationship.
- **Channels are bidirectional but control-gated.** All channels produce and consume events. Control signal authority requires explicit enablement (default: disabled).
- **Everything is permission-based.** Domains, tools, sensors, control signals — all access is gated by entity permissions enforced by the kernel.

### 1.5 Entity Model

An **Entity** is any persistent thing that can participate in events. Examples include internal agent personas, administrative users, general users, pets, physical objects, locations, and groups.

Entities are lightweight graph nodes with relationships and metadata, not containers for isolated memories. The runtime maintains a single immutable history and a unified knowledge graph from which multiple entity-specific projections are derived.

```
Entity
├── Identity
├── Type (internal | external)
├── Trust Level (admin | admin_delegate | trusted_user | sub_user | untrusted)
├── Relationships
├── State (projection of history)
└── Permissions (retrieval policy)
```

**Internal entities** represent operational projections of the runtime (Technical Assistant, Companion, Stream Host, Automation Agent). They define retrieval policies over the shared graph — accessible domains, privacy levels, allowed channels, tool permissions, and sensor permissions. Internal entities do not require Admin elevation.

**External entities** represent observed participants in the environment. They are constructed from history and graph relationships, starting as untrusted with no access. Admin or admin_delegate must explicitly elevate them and grant permissions.

The Admin entity exists from system bootstrap, pre-linked to the primary webUI session. It is the single root of trust and cannot be demoted or superseded.

→ See [ENTITIES.md](./core/ENTITIES.md) for the full entity model specification.

### 1.6 Implementation Note: Two Stores, Seven Graphs

The seven logical graphs are split across two data stores, reflecting two fundamentally different data models.

**World-State Event Store** — append-only, content-addressed events. State is derived by projecting event history. Four graphs share this store:

```
World-State Event Store (append-only, content-addressed)
    │
    ├── Perception projection  ── what happened
    ├── Execution projection   ── what was done
    ├── Memory projection      ── what was experienced
    └── Knowledge projection   ── what is known
```

**Artifact Version Store** — content-addressed, semantically versioned, lifecycle-managed. State is the latest version per artifact. Three graphs use this store:

```
Artifact Version Store (content-addressed, semantically versioned)
    │
    ├── Macro Graph     ── compiled execution patterns
    ├── Skill Registry  ── pre-authored behavior definitions
    └── Code Registry   ── tested code components
```

Each store maintains its own indexing strategy, retention policies, and query patterns. Cross-store references use content hashes in events pointing to artifact ID+version — no unified index is needed. An event in the world-state store references an artifact like `macro_used: macro_weather_check:v3.1.0`. An artifact in the version store references events for provenance like `provenance: { source_events: [evt_abc, evt_def, ...] }`.

The separation reflects that world-state graphs are projections of lived experience (events), while artifact graphs are versioned entities the system creates, manages, and evolves over time.

→ See [GRAPH.md](./core/GRAPH.md#one-line-architecture-summary) for visual diagrams and design review heuristics.

### 1.7 Cold-Start Problem

The system improves over time through accumulated structure, but the initial period — when there is no memory, no knowledge, and no macros — requires specific design attention. **Skills are an optional acceleration; the system works without them.**

**Two paths through the cold-start period:**

**With skills (optional):** The system executes pre-authored skills through the RPU, providing structured, competent behavior immediately. Skills are not compressed at this stage — they require full reasoning — but they eliminate the "no behavior at all" problem. Memory and knowledge provide context that makes skill execution more effective. As skills execute, their traces enter the event graph, ready for the Phase 5 compression pipeline.

**Without skills (default):** The system operates as a structured but uncompressed runtime. Every task requires full reasoning. The RPU is invoked for all interpretation, planning, and summarization. This is functionally equivalent to a well-engineered prompt-chain agent with structured contracts. Memory and knowledge accumulate organically from the system's own execution. Patterns are discovered from general execution traces in Phase 5. Slower start, but fully functional.

**Bootstrapping process (with skills):**

1. **Phase 4 Day 1:** The system loads installed skills (if any) and executes them through the RPU. Skills provide competent behavior in their domains; default personality parameters handle everything else. Memory and knowledge systems are active but empty.
2. **Phase 4 Week 1:** Narrative memories begin accumulating. Skill execution traces (or general execution traces if no skills) provide the first structured data for macro discovery.
3. **Phase 4 Month 1:** Validated knowledge begins entering the knowledge graph. Temporal patterns emerge from memory chains. Skills now execute with meaningful context from memory and knowledge.
4. **Phase 5 Month 3+:** The compression loop is operational. Skill-derived macros handle frequent skill-based tasks deterministically. Pattern-derived macros handle general repeated behaviors. Knowledge eliminates re-derivation. The system's reasoning cost per task begins to measurably decrease.

**Cold-start mitigation strategies:**

- **Skills in existing harness format (optional)** provide competent behavior from Phase 4. Users install skills they trust; the system executes them with full reasoning. No new format to learn, no migration required. Skills can be installed at any time — during initial setup, or later as the user discovers new needs. The system may also suggest skills based on observed behavior patterns that match available skill capabilities.
- **Default personality parameters** provide reasonable behavior for tasks not covered by installed skills. The personality evolves as experience accumulates.
- **Pre-built knowledge templates** (optional) can bootstrap common knowledge domains (user preferences, common facts) to accelerate early learning.
- **User onboarding** can explicitly teach the system key preferences and patterns, jump-starting the knowledge graph.
- **The evaluation baseline (Phase 2) is measured during the cold-start period.** This ensures that the thesis is tested from the worst-case starting point, not from a pre-loaded state. Skills are not part of the thesis test.

The cold-start period is not a flaw — it is the natural consequence of a system that learns from experience. Skills accelerate the early phase but do not change the fundamental learning curve. The evaluation methodology (Section 16) measures improvement from this baseline, not from an idealized pre-loaded state.

**Residual gap:** Skills only cover tasks their authors anticipated. When no skill exists for a task category, the system falls back to general RPU reasoning until it discovers patterns through its own execution. Skill coverage is a user decision — the system does not impose a skill catalog.

---

## 2. Related Work

Byte draws from and differs from several established research areas:

**Event Sourcing** (Fowler, 2005) provides the append-only event log model. Byte extends this by treating cognitive events — perception, reasoning steps, intent proposals — as first-class events alongside system events, and by defining projections that derive state across multiple cognitive domains.

**Cognitive Architectures** such as SOAR (Newell, 1990) and ACT-R (Anderson, 1993) model human cognition through production rules and memory systems. Byte differs in that it does not attempt to model human cognition; it uses AI reasoning as a replaceable transform within a deterministic runtime, and focuses on reducing reasoning cost through structural accumulation rather than modeling cognitive processes.

**Agent Frameworks** like LangChain, AutoGen, and CrewAI orchestrate LLM calls through prompt chains and tool-use patterns. Byte replaces this model entirely: the LLM is a coprocessor with structured contracts, not the control flow. Execution is managed by a deterministic kernel, not by the model's output.

**Knowledge Graphs** provide structured fact storage and semantic query. Byte's knowledge graph differs by carrying temporal semantics (validity windows, confidence decay, revision chains) and by deriving knowledge from an event-sourced pipeline rather than manual curation.

**Git and Version Control** provide the content-addressed, DAG-structured, immutable history model. Byte applies this to cognitive events rather than source code, and adds projection layers that derive current state from history.

**Effect Systems** (e.g., Effect.ts) provide the semantic orchestration primitives. Byte adopts these as a language-agnostic execution meaning layer, not tied to any specific runtime.

---

## 3. Architecture Overview

Byte is a persistent, distributed, event-sourced runtime where cognition, perception, and action are unified through a world-state graph. Everything — LLM inference, sensor inputs, tool calls, streaming, automation — becomes a projection or transformation of that graph.

The system treats AI reasoning as a replaceable transform, not the center of the architecture. It can host chatbots, agents, automation systems, multimodal interfaces, and distributed edge nodes — but none of those define the architecture. The kernel, event graph, scheduler, and execution model carry the accumulated structure. Models come and go; the runtime persists.

### 3.1 System Layers

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

The stores hold all system state — events (perception, execution, memory, knowledge) and artifacts (macros, skills, code components). The kernel governs execution, invoking the RPU for structured reasoning and semantic orchestration primitives for deterministic tool composition. The optimization layer compresses experience into reusable structure. Channels expose projected state to external surfaces.

This diagram shows the homelab runtime. Edge nodes run perception processing and local inference, sending only structured events to the homelab. AI models are accessed through the LLM Runtime adapter and are not part of the core architecture.

### 3.2 Build Phases and Document Mapping

| Phase | Component                           | Document Sections | Core Documents                                | Status          |
| ----- | ----------------------------------- | ----------------- | --------------------------------------------- | --------------- |
| 1     | Kernel + Execution Graph            | Sections 4, 5     | GRAPH.md                                      | Design complete |
| 2     | RPU + Orchestration                 | Sections 6, 10    | RPU.md, ORCHESTRATION.md                      | Design complete |
| 3     | Signal-to-Intent Pipeline           | Section 7         | —                                             | Design complete |
| 4     | Memory + Knowledge + Skill Registry | Section 8, 9.3    | MACROS.md (skill execution)                   | Design complete |
| 5     | Macros + Code Registry              | Section 9.1, 9.2  | MACROS.md, REGISTRY.md                        | Research phase  |
| 6     | Edge + Multimodal                   | Sections 12, 13   | EDGE_ARCHITECTURE.md, MULTIMODAL_INTERFACE.md | Conceptual      |

Running example: Section 11. Evaluation: [EVALUATION.md](core/EVALUATION.md).

Phases 1–4 test the core thesis. Phase 4 (memory + knowledge) is the falsification point. Skills are a harness completeness feature available from Phase 4 but not part of the thesis test. Phase 5 adds the optimization layer: compiling skill execution traces and general patterns into deterministic macros. Phases 5–6 are experimental extensions.

---

## 4. World-State Graph Architecture

The world-state graph is the single source of truth for the entire system. It is an event graph where every piece of system activity is recorded as a structured event with normalized arguments, timestamps, and dependency edges.

### 4.1 Sub-Graph Architecture

Conceptually the world-state is unified. Operationally the system uses seven logical graphs split across two data stores (Section 1.5). Each graph handles a different domain.

#### 4.1.1 World-State Graphs (Event Store)

These four graphs are projections of the append-only event stream. Current state is derived by replaying events.

**Perception Graph** — structured perception from signal processing systems, environmental facts, user state, biometrics. Answers "what is happening in the world." Contains the outputs of object detection, speech-to-text, attention tracking, movement analysis, and cognitive state processing. These are the first entries in the graph — raw sensor streams never enter.

**Execution Graph** — chains, tasks, scheduling decisions, tool calls, RPU invocations. Answers "what is the system doing." Contains execution chain lifecycle, scheduler decisions, tool call records, RPU request/response pairs, and state transitions.

**Memory Graph** — episodic experiences, emotional context, narrative memories, personality evolution, relationship state. Answers "what has been experienced." Retrieved by similarity, temporal proximity, and emotional valence. Evolves as new experiences accumulate and context shifts.

**Knowledge Graph** — validated facts, semantic knowledge, tested relationships, established patterns. Answers "what is known to be true." Retrieved by semantic query. Facts must pass a validation pipeline before becoming knowledge. Knowledge carries temporal validity, confidence decay, and revision chains — it is durable but not final. Also serves as the source for temporal intent generation when knowledge carries explicit temporal patterns.

#### 4.1.2 Artifact Graphs (Version Store)

These three graphs are versioned entities with semantic versioning and lifecycle management. Current state is the latest version per artifact.

**Macro Graph** — compiled execution subgraphs. Discovery is passive through sliding window mining over execution traces and active through skill execution trace analysis. Validation tests against historical traces. Macros may be skill-derived (carrying provenance links to their source skill) or pattern-derived (discovered from general execution). Macros may compose child macros hierarchically. Answers "what patterns repeat and whether they are valid."

**Skill Registry** — pre-authored behavior definitions in the existing harness format (instruction files + tool references). Content-addressed, versioned with semantic tags. Skills are not executed differently from other behavior — they generate execution traces that feed the compression pipeline. The Skill Registry answers "what behaviors are available as seeds." Skills come from any source; the user decides what to trust.

**Code Registry Graph** — versioned, tested code components that the model and agents draw from when writing software with users. Not executed by the runtime; used to compose new programs. Components are actively authored by the model, agents, or the user. A mandatory test pipeline validates before adoption. Answers "what code exists, what version is current, and whether it passes tests."

### 4.2 Cross-References Between Sub-Graphs

Graphs reference each other through content-addressed hashes. Within the world-state event store, references work like git submodules — content-addressed hashes link components across domains while keeping queries simple. A perception (dog detected) can be referenced by execution (chain triggered object interaction). An execution outcome (task completed) can be referenced by memory (experience indexed). A memory lookup (recall past interaction) can be referenced by execution (context injected into RPU request). A validated fact (Bob prefers Rust) can be referenced by execution (code generation chose Rust for Bob's project).

**Cross-store references** connect the world-state event store to the artifact version store:

- **Events reference artifacts.** When an execution chain uses a macro, the event records `macro_used: macro_weather_check:v3.1.0`. When a skill is installed, the event records `skill_installed: skill_morning_briefing:v1.2.0`. The content hash in the event points to the artifact ID+version.
- **Artifacts reference events for provenance.** A macro carries `provenance: { source_events: [evt_abc, evt_def, ...] }` linking it to the execution traces it was compiled from. A skill-derived knowledge entry carries `provenance: { source_skill: "skill_morning_briefing", source_version: "v1.2.0", source_events: [...] }`.

No unified index is needed. Each store maintains its own indexing. Cross-store resolution is a two-step lookup: event content hash → artifact ID+version → artifact store lookup.

### 4.3 Events as Immutable Commits

Every event in the world-state event store is immutable. Events reference their causal parents, are cryptographically hashed and optionally signed, and the history is append-only. This creates a tamper-evident record of all system activity.

Artifacts in the version store follow a different model: they are versioned entities with semantic versioning and lifecycle states (created, promoted, demoted, superseded, deprecated). Individual artifact versions are immutable once published, but new versions can be created. The artifact store maintains the full version history for each artifact.

### 4.4 State as Projection

For world-state graphs, current state is derived from replaying events. State is a materialized view, not the source of truth. Checkpoints accelerate reconstruction but do not replace history. The system can always rebuild current world-state from the full event log.

For artifact graphs, current state is the latest version of each artifact. The artifact store maintains version metadata (current version, lifecycle status, semantic tags) that determines which version is active. Artifact versions are immutable once published; state changes occur through version creation, not modification.

### 4.5 Projections and Channels

The runtime distinguishes between two categories of transformation over the event graph:

**Projections** transform graph state into graph state. They are internal runtime transformations that produce durable, queryable, composable knowledge structures. A projection is a pure function over an event stream:

```
Graph → Projection → Graph
```

Examples:

```
Perception → Perception Graph          (world-state event store)
Perception Graph → Situation Model     (world-state event store)
Situation Model + User Signals → Intent (world-state event store)
Intent → Execution Graph               (world-state event store)
Execution Graph → Memory Graph         (world-state event store)
Memory Graph → Knowledge Graph         (world-state event store)
Execution Graph → Macro Graph          (artifact version store, via discovery pipeline)
Skill Execution → Execution Graph → Macro Graph (artifact version store, skill-derived)
Skill Execution → Execution Graph → Knowledge Graph (world-state event store, skill-derived)
```

Projections that write to the world-state event store produce events. Projections that write to the artifact version store produce artifact versions. The discovery pipeline (Section 9.1) reads from the world-state event store and writes to the artifact version store when proposing new macro versions.

Projections optimize for correctness, queryability, persistence, and composition. They can be chained — the output of one projection becomes the input of another. The summarization pipeline (section 8) is a chain of projections: situation model into narrative memories, into validated knowledge.

**Channels** transform graph state into external representation and translate external inputs back into events. They are bidirectional boundaries between the runtime and the outside world. All channels are bidirectional — they both produce events into the system and consume projected views from it. The primary orientation (input vs. output) determines the dominant flow, but both directions are always available.

```
Graph → Channel → External Surface
External Surface → Channel → Event
```

Channels must be approved by Admin (or admin_delegate with `approve_channels` permission) before they process any events. Unapproved channels exist in `pending_approval` status with no event processing capability. Channels are not control channels by default — even Admin-connected channels must be explicitly enabled to send kernel control signals.

The event surface of a channel is analogous to a tool definition: it declares what it produces and what it consumes.

Examples:

```
Execution Graph → Web UI Channel
Execution Graph → Discord Channel
Execution Graph → CLI Channel
Memory Graph → Search API Channel
Execution Graph → Telemetry Channel
Perception Graph → Streaming Channel
Camera Channel → Perception Events (input-oriented)
Microphone Channel → Perception Events (input-oriented)
```

Channels optimize for rendering, filtering, sorting, formatting, and aggregation. Channels are replaceable and independent; the same projection graph can serve any number of channels without conceptual changes. Channels are responsible for their own caching and state aggregation — logically both sides see events, but channels may aggregate to present a "current state" view.

The distinction matters because projections build runtime knowledge while channels consume it. Projections are durable; channels come and go. The browser, Discord, voice interfaces, and the wearable edge node are all channels attached to the same projection graph.

**Decision rules:** If a transformation doesn't produce durable runtime knowledge, it's a channel, not a projection. If a channel tries to produce runtime state, it should emit events instead. If a new subsystem doesn't fit naturally into this model, it's introducing a second source of truth.

→ See [CHANNELS.md](./core/CHANNELS.md) for the full channel specification: approval workflow, entity binding, control signals, bidirectional flow.
→ See [GRAPH.md](./core/GRAPH.md#the-full-projection-pipeline) for visual diagrams of the projection and channel flow.

### 4.5.1 Projection Types

Projections are classified by their determinism level. This classification governs how they are debugged, replayed, and versioned.

**Deterministic projections** produce identical output given identical input. They are fully replayable, versioned by their logic hash, and form the backbone of kernel-level state derivation. Examples: perception processing (object detection, speech-to-text), execution graph derivation, chain lifecycle state projection.

**Probabilistic projections** produce output that may vary across replays due to LLM involvement. They carry confidence scores, are versioned by both logic hash and model version, and require evidence trails for debugging. Examples: situation model generation, intent estimation from fused signals.

**Hybrid projections** combine deterministic and probabilistic stages. The deterministic stages are fully replayable; the probabilistic stages carry confidence and model version metadata. Examples: summarization pipeline (deterministic aggregation + LLM-assisted narrative generation), knowledge validation (deterministic corroboration + LLM-assisted contradiction detection).

This classification matters for debugging. When state is incorrect, the projection type determines the investigation path: deterministic projections are debugged by replaying input; probabilistic projections are debugged by examining evidence trails and confidence scores; hybrid projections are debugged by isolating which stage produced the error.

### 4.6 DAG Structure

The graph is a DAG, not a linear chain. Multiple chains execute concurrently. Reasoning can fork. Execution paths can merge. Lineage is preserved across all branches. Execution chains behave like branches — long-running tasks become branches of execution that can pause, resume, fork, merge, or terminate. The scheduler determines which branches receive resources.

### 4.7 Provenance

Provenance is first-class. Every action is traceable to its origin: perception processing leads to world-state version, which leads to situation model generation, which leads to intent proposal, which leads to scheduler decision, which leads to tool execution. The system can answer what happened, why it happened, what information was available, what alternatives existed, and which chain caused it.

### 4.8 Git, Not Blockchain

The goal is not distributed consensus. The goal is tamper-evident history, causal lineage, replayability, auditability, and reproducibility. The architecture is closer to Git plus Event Sourcing plus Knowledge Graph plus Distributed Runtime than to a blockchain.

### 4.9 Entity Model and Memory Scoping

An **Entity** is any persistent thing that can participate in events. Entities are lightweight graph nodes with identity, relationships, and permissions — not memory containers. All memory lives in the shared graph with metadata describing context.

**Internal entities** (Technical Assistant, Companion, Stream Host, Automation Agent) are operational projections of the runtime. They define retrieval policies over the shared graph: accessible domains, privacy levels, allowed channels, tool permissions, and sensor permissions. Internal entities do not own separate memory stores.

**External entities** (Admin, family members, guests, pets, devices) are constructed from history and graph relationships. They start as untrusted with no access. Admin or admin_delegate must explicitly elevate them and grant permissions.

The Admin entity exists from system bootstrap, pre-linked to the primary webUI. It is the single root of trust and cannot be demoted or superseded.

Memory does not belong to an entity. Instead, it carries metadata describing its context:

```
Memory
├── Subject(s) — which entities are involved
├── Origin Channel — where it came from
├── Topic — what it concerns
├── Privacy Classification — inferred from who and what
├── Relationship Context — which relationship it exists within
└── Domain Tags — technical, project, research, interaction, relationship, personal, private, credentials
```

Privacy is inferred from the structure of memory: who is involved and what the memory concerns. This information becomes part of memory metadata and survives abstraction into knowledge.

Knowledge carries dual access levels: factual content (globally accessible when domain permissions allow) and contextual metadata (scoped to the originating relationship). This prevents accidental leakage of contextual information while allowing useful abstractions to propagate.

Retrieval is the intersection of multiple filters:

```
Candidate Memory
    ∩ Entity Policy (accessible domains, privacy levels)
    ∩ Channel Policy (allowed/blocked domains, privacy ceiling)
    ∩ Memory Domain (task-relevant domains)
    ∩ Privacy Rules (privacy level <= effective ceiling)
    ∩ Task Relevance (semantic similarity to current task)
    = Active Context
```

→ See [ENTITIES.md](./core/ENTITIES.md) for the full entity model: schema, trust levels, lifecycle, permissions.
→ See [RETRIEVAL.md](./core/RETRIEVAL.md) for the retrieval pipeline: filter chain, dual-access projection, RPU integration.

---

## 5. Cognitive Runtime Kernel

The kernel is the irreplaceable core of the architecture. It is the permanent component — the part that cannot be swapped out or replaced without rebuilding the entire system. While the RPU can be exchanged for any reasoning engine, the kernel governs all execution.

It manages execution chains as DAG-based computation graphs rather than linear sequences. It handles scheduling, prioritization, interruption and resumption, long-lived workflows, resource arbitration, and tool execution lifecycle.

The kernel receives structured intent proposals from the LLM and other sources, validates them against current world-state, schedules them according to priority and resource availability, and executes them using the semantic orchestration primitives.

### 5.1 Non-Graph Runtime Layer

The scheduler, execution workers, cache system, and resource arbitrator are not part of the world-state graph. They operate over the graph, not inside it. These components are ephemeral runtime infrastructure — they manage execution flow, allocate compute, and maintain in-memory state for active chains. They do not produce graph entries directly; they produce execution outcomes that become events in the execution graph.

This distinction matters for debugging and system boundaries. The world-state graph is the persisted, replayable record of everything that happened. The non-graph runtime layer is the machinery that makes things happen — it is transient, replaceable, and not subject to replay. If the kernel restarts, the non-graph layer rebuilds from the current graph state.

### 5.2 Execution Model

The system runs on a unified abstraction: an execution chain is a persistent, prioritized, interruptible process.

Each chain persists across time, survives waiting and interruption, is scheduled dynamically, can spawn subchains, can be resumed after suspension, and is governed by priority and resource quotas.

Chains are not sequences of prompts or tool calls. They are long-lived chains of computation that may include reasoning steps, tool execution, waiting for external events, user interaction, retries and recovery flows, and nested subchains.

The scheduler operates on execution opportunities, not queue draining. It continuously allocates inference and tool capacity across competing chains based on priority, resource availability, and current world-state.

#### 5.2.1 Chain Lifecycle States

- **Active** — currently executing
- **Waiting** — paused for an external event or signal
- **Parked** — suspended due to resource constraints, lower priority
- **Resumed** — restored from waiting or parked state
- **Escalated** — priority elevated due to dependency or user intervention

#### 5.2.2 Priority Tiers

- **Critical** — safety, security, system integrity. Non-interruptible.
- **Interactive** — user-facing responses. High priority, interruptible by critical.
- **Automation** — background workflows, tool chains. Interruptible by critical and interactive.
- **Background** — indexing, optimization, non-urgent tasks. Interruptible by all higher tiers.

#### 5.2.3 Branch Semantics

Execution chains behave like branches in a version control system. They can fork when reasoning diverges, merge when execution paths converge, and terminate when complete. The scheduler manages resource allocation across all active branches, ensuring that critical chains receive priority while background work proceeds when capacity allows.

Key responsibilities:

- Execution chain lifecycle management with states including active, waiting, parked, resumed, and escalated
- Priority-based weighted scheduling across critical, interactive, automation, and background tiers
- Dynamic repartition of compute resources between runtime execution and offline optimization
- Priority inheritance across full execution chains so that subchain urgency propagates correctly
- Hard non-interruptibility for critical safety and security chains
- Interruptible inference for non-critical chains only
- Resource arbitration across competing chains, sub-graphs, and edge nodes

### 5.3 RPU Integration

The kernel invokes the Reasoning Processing Unit as a specialized coprocessor, not as an autonomous agent. The RPU receives structured requests containing world-state projections, personality state, context, and task definitions. It returns structured results — plans, summaries, state update proposals, and action proposals — rather than raw conversational text.

### 5.4 Planning-First Execution

Long-running tasks follow a planning-first workflow. The user request triggers an acknowledgement, then plan generation by the RPU, then execution, then a recap, then the conversation response. The plan becomes an observable runtime artifact. Execution updates the plan state rather than generating arbitrary conversational messages. This creates deterministic visibility into system progress.

Plan artifacts are recorded in the execution graph alongside tool calls and reasoning steps. When a planning-first execution chain repeats, the plan artifact becomes part of the captured macro subgraph. When the repetition is at a lower level (tool calls only), the macro captures only that subgraph. The macro does not mandate what is captured — it compiles whatever the discovery pipeline detects as the repeating pattern.

### 5.5 Recap-Based Observability

The runtime maintains execution state independently from the conversation. Users may request current status, task progress, plan state, or execution recap. The response is generated from runtime projections rather than reconstructed from conversation history. This avoids using conversational messages as the source of truth.

---

## 6. Reasoning Processing Unit

The RPU treats a Large Language Model as a specialized reasoning coprocessor rather than as a complete autonomous agent. Instead of embedding memory, planning, identity, world state, execution management, and communication into a single prompt, these concerns are externalized into dedicated runtime systems. The model becomes a transformation engine operating over structured state.

The objective is not to maximize model intelligence, but to minimize the amount of reasoning required to produce intelligent behavior. Intelligence is the ability to accumulate structure so that less reasoning is required in the future.

### 6.1 RPU Applied to Perception

The same RPU pattern governs situation model generation in the signal-to-intent pipeline. The model receives structured perception (JSON) along with context and personality state, and produces structured situation model output (JSON). Fixed input schema, fixed output schema, no free-form text. The model only ever sees JSON in → JSON out, whether interpreting perception or performing reasoning.

### 6.2 RPU Responsibilities

The RPU is responsible for: interpretation, reasoning, synthesis, planning, summarization, reflection, communication generation, and cognitive transformations.

The RPU is not responsible for: memory persistence, event storage, scheduling, state tracking, personality storage, tool orchestration, world modeling, or execution management. These functions belong to the runtime.

### 6.3 Cognitive Contract

Every inference executes through a structured contract. The RPU receives a request containing the function to perform, the objective, personality state, world state, context projection, and optionally task state and previous artifacts. It returns a result along with optional summary, plan, memory suggestions, state updates, next actions, and metadata including confidence and reasoning mode.

```typescript
interface RPURequest {
  function: string;
  objective: string;
  personality: PersonalityState;
  worldState: WorldState;
  context: ContextProjection;
  taskState?: TaskState;
  previousArtifacts?: Artifact[];
}

interface RPUResponse {
  result: unknown;
  summary?: string;
  plan?: Plan;
  memorySuggestions?: MemoryEvent[];
  stateUpdates?: StateUpdate[];
  nextActions?: ActionProposal[];
  metadata?: {
    confidence?: number;
    reasoningMode?: string;
  };
}
```

The RPU produces structured results rather than raw conversational text.

### 6.4 Design Philosophy

Traditional agent architectures embed everything into a single prompt:

```
Conversation + System Prompt + Tools + Memory Instructions + Personality Instructions = Agent
```

The RPU model separates concerns:

```
Runtime State + World State + Personality State + Task Definition + Reasoning Function = Cognitive Result
```

The agent is not the model. The agent emerges from the interaction between runtime subsystems and reasoning functions.

### 6.5 Personality as State

Personality is stored as structured data rather than prompt text. The trait-vector example below illustrates the form this takes, but the full personality model is richer:

```json
{
  "curiosity": 0.9,
  "humor": 0.4,
  "technicalDepth": 0.95,
  "directness": 0.8
}
```

Personality is composed of traits, preferences, communication style, values, relationship state, and interaction history. All of these are projections of accumulated events in the memory graph — personality evolves as the system accumulates experience, not through manual configuration.

The runtime owns personality. The RPU consumes personality projections. This allows model replacement, personality versioning, personality evolution, and consistent behavior across backends.

### 6.6 Cognitive Cache Hierarchy

The architecture operates as a hierarchy of increasingly expensive cognitive operations:

- L1 — Active Context
- L2 — Structured State
- L3 — World Model
- L4 — Event History
- L5 — Reasoning (RPU)

Reasoning is the most expensive resource. The purpose of the runtime is to convert reasoning into reusable structure and prevent unnecessary recomputation.

### 6.7 Guiding Question

The central design question of the RPU architecture is: "What can be removed from the reasoning loop?" Any responsibility that can be represented as deterministic state, stored structure, or runtime infrastructure should be externalized from the model. The RPU should only perform transformations that genuinely require reasoning.

### 6.8 Model Independence

The RPU abstraction enables model interchangeability. The runtime defines state, contracts, events, personality, memory, and scheduling. The model only implements reasoning functions. This allows multiple model classes to participate in the same cognitive runtime.

### 6.9 LLM Runtime

A standalone adapter layer that standardizes heterogeneous LLM providers into a universal runtime representation. It adapts any model — OpenAI-compatible, native APIs, or custom backends — to the same normalized IR schema.

Responsibilities include normalizing streaming inference outputs, translating tool-call formats across providers, preserving reasoning and structured outputs when available, and decoupling providers from downstream systems.

The LLM Runtime ensures that the cognitive runtime kernel never needs to know which specific provider is backing a given inference request. All providers emit the same IR format. This layer is about model adaptation only — it does not use semantic code components or orchestration primitives.

→ See [RPU.md](./core/RPU.md#cognitive-flow-diagrams) for cognitive flow diagrams and the "agent ≠ model" framing.

---

## 7. Signal-to-Intent Pipeline

The pipeline converts raw signals into actionable intent through three layers. Each layer produces structured data that is recorded in the event log for provenance, model training, and system improvement.

### 7.1 Perception Processing (Signal → Structure)

Specialized systems convert raw sensor streams into structured perception. This is the first layer of the pipeline and the source of all graph entries.

Camera frames are processed by object detection models (YOLO-style) that produce perception with labels, bounding boxes, and confidence scores. Audio streams flow through speech-to-text systems that produce transcription perception. Gaze samples are processed by attention trackers that produce fixation perception with targets and durations. IMU readings are analyzed by movement pattern systems that produce locomotion and gesture perception. EEG signals are processed by cognitive state systems that produce attention, fatigue, and engagement indices.

Perception processing is deterministic and model-based, not LLM-based. It runs on edge devices for low-latency perception and on the homelab for heavier models. It reduces sensor frequency (30 fps, 100 Hz) to perception frequency (~1-10/sec per sensor).

All perception carries confidence scores and is recorded in the perception graph. Because perception is preserved, all downstream interpretation can be reprocessed with better models as they become available.

Example perception from camera processing:

```json
{
  "type": "perception",
  "modality": "vision",
  "timestamp": "2024-01-01T12:00:00Z",
  "objects": [
    { "label": "laptop", "confidence": 0.94, "bbox": [120, 80, 340, 260] },
    { "label": "mug", "confidence": 0.87, "bbox": [400, 200, 450, 280] }
  ],
  "scene": { "room": "kitchen", "lighting": "indoor" },
  "source": "camera_front",
  "confidence": 0.91
}
```

### 7.2 Situation Model Generation (Structure → Meaning)

Situation model generation combines multiple perception types into coherent understanding of what is happening.

Situation model generation runs on the homelab and uses the RPU pattern: structured JSON input produces structured JSON output. The model receives batched windows of perception (e.g., 30-second windows) along with context and personality state, and produces situation model output with confidence scores and evidence trails.

The model does not generate free-form text. It receives a deterministic prompt with a fixed output schema and returns structured situation model output. This is the same RPU contract used for reasoning, applied to perception interpretation.

Situation model generation is LLM-assisted. Without it, the system falls back to rule-based heuristics with significantly reduced fidelity. Perception processing continues regardless — the system degrades but does not fail.

Example situation model:

```json
{
  "type": "situation_model",
  "interpretation": "user is in focused work session on laptop",
  "confidence": 0.85,
  "evidence": {
    "objects": ["laptop", "mug"],
    "attention": { "target": "laptop", "duration_ms": 4200 },
    "cognitive_load": 0.7
  },
  "sources": ["per_abc123", "per_def456", "per_ghi789"],
  "validation_status": "hypothesis"
}
```

Situation model output is recorded in the event log but is an intermediate transform — it feeds into intent estimation rather than becoming a destination graph.

### 7.3 Intent Estimation (Meaning → Action)

Intent estimation converges three sources into a unified intent format:

**User intent** — explicit signals from the multimodal layer. Voice provides explicit commands. Gaze provides attention and selection. EMG and blink provide confirmation. EEG provides state modulation. These signals are fused into user intent.

**System intent** — autonomous proposals generated from situation model and context. When situation model indicates the user has been stuck on a problem for 30 minutes (focused work session + increasing cognitive load), the system may propose offering help. When situation model indicates the user is cooking, the system may propose pulling up a recipe. These are system-generated intent proposals.

**Temporal intent** — scheduled intent generated from knowledge entries that carry temporal patterns. When knowledge indicates a recurring behavior ("user checks weather daily at 7 AM") or a one-time future event ("remind me tomorrow at 3 PM"), the temporal intent generator produces intent at the appropriate time. These are proactive, not reactive — they don't wait for a situation model trigger. High-confidence knowledge fires automatically; user denial of automated help is recorded as evidence that feeds back into knowledge, adjusting confidence or invalidating the pattern.

All three sources converge into the same intent format. Intent flows through three stages before execution:

**Intent** — a state change request or desire. Originates from user signals or system-generated situation model interpretation. Carries confidence, context, and evidence trails. Does not specify how to execute.

**Proposal** — an executable candidate derived from intent. The kernel or RPU translates intent into a concrete action plan with specific tool calls, chain triggers, or state updates. Proposals are validated against current world-state before proceeding.

**Commit** — a kernel-approved execution decision. The scheduler validates the proposal against resource availability, priority constraints, and safety rules. Once committed, the proposal becomes an execution chain in the execution graph.

This three-stage flow separates concerns: intent expresses what is desired, proposal specifies how to do it, commit authorizes execution. The kernel does not distinguish between user-originated, system-originated, or temporally-originated intent at the intent stage — all flow through the same validation, proposal generation, and commit pipeline.

Example intent:

```json
{
  "type": "intent",
  "source": "system",
  "desire": "offer assistance with current task",
  "confidence": 0.78,
  "triggered_by": "sm_xyz789",
  "context": {
    "situation_model": "user is in focused work session",
    "duration_minutes": 30,
    "cognitive_load_trend": "increasing"
  }
}
```

Example proposal (derived from intent):

```json
{
  "type": "proposal",
  "intent_ref": "int_abc123",
  "actions": [
    {
      "type": "rpu_invoke",
      "function": "generate_assistance_prompt",
      "context": "current_task_summary"
    },
    { "type": "channel_send", "channel": "web_ui", "message_ref": "pending" }
  ],
  "priority": "interactive",
  "resource_estimate": { "tokens": 2000, "duration_ms": 5000 }
}
```

Example commit (kernel decision):

```json
{
  "type": "commit",
  "proposal_ref": "prp_def456",
  "chain_id": "chain_789",
  "approved_actions": ["rpu_invoke", "channel_send"],
  "priority": "interactive",
  "resource_quota": { "tokens": 2000, "timeout_ms": 10000 }
}
```

### 7.4 Reprocessability

Because perception is preserved in the event log, the entire downstream pipeline can be reprocessed. Situation model generation can be re-run with better models. Intent estimation can be re-evaluated with improved context understanding. This enables continuous system improvement without losing historical data.

The offline optimization loop (section 14) reprocesses historical perception to generate improved situation model and intent, which updates the memory and knowledge graphs without modifying the immutable perception log.

### 7.5 Perception Uncertainty Propagation

Perception processing uses probabilistic models (object detection, speech-to-text, attention tracking) that produce outputs with confidence scores. This uncertainty propagates through the entire pipeline:

**Perception → Situation Model:** The situation model generator receives perception with confidence scores and produces a situation model with its own confidence. Low-confidence perception reduces situation model confidence proportionally. Multiple independent perception sources confirming the same observation increase confidence (Bayesian evidence accumulation).

**Situation Model → Intent:** Intent estimation combines situation model confidence with user signal confidence and temporal knowledge confidence. The fused confidence determines whether the intent is executed, suggested, or logged (see Section 13 threshold mechanics).

**Decision thresholds under uncertainty:**

| Perception Confidence | Situation Model Confidence | Intent Confidence  | Kernel Action                       |
| --------------------- | -------------------------- | ------------------ | ----------------------------------- |
| High (>0.9)           | High (>0.85)               | High (>0.9)        | Execute immediately                 |
| High                  | Moderate (0.7–0.85)        | Moderate (0.7–0.9) | Present as suggestion               |
| Moderate (0.7–0.9)    | Any                        | Low (0.5–0.7)      | Log as context                      |
| Low (<0.7)            | Any                        | Any                | Discard; flag for model improvement |

**Cascading uncertainty:** A single low-confidence perception does not necessarily block downstream action if other modalities provide strong corroborating evidence. The system uses multi-modal fusion to compensate for individual modality weakness. However, if all contributing perceptions are low-confidence, the downstream intent is automatically capped at the "log as context" level regardless of situation model output.

**Uncertainty tracking:** Every event carries the confidence scores of its upstream dependencies. This enables post-hoc analysis: "Why did the system make this decision?" can be answered with "Because perception X had 0.94 confidence, which produced situation model Y with 0.88 confidence, which produced intent Z with 0.91 confidence."

---

## 8. Event Summarization and Indexing

The signal-to-intent pipeline (section 7) produces perception, situation model, and intent at high frequency. The summarization pipeline operates downstream of intent, converting accumulated experience into progressively more compact and meaningful representations suitable for long-term storage and retrieval.

Without summarization, the system would drown in accumulated experience. The summarization pipeline converts high-volume situation model into progressively more compact narrative memories and validated knowledge.

### 8.1 Summarization Pipeline

The event log is the complete, unfiltered record of all perception, situation model, intent, execution, and system state transitions. This layer is append-only and immutable. It is the source of truth but not a query surface.

Narrative memories are consolidated experiences indexed for long-term recall. A sequence of situation model from a trip becomes "visited location Y, encountered situation Z, took action W, outcome was positive." Narrative memories carry temporal and contextual metadata — frequency, timing, conditions — that enables temporal pattern extraction during knowledge validation. Narrative memories populate the memory graph and serve as context for future RPU invocations.

Every memory carries explicit metadata describing its context:

```
Memory
├── content: the memory content
├── subjects: [entity_id, ...]           // which entities are involved
├── origin_channel: channel_id            // where it came from
├── topic: string                         // what it concerns
├── privacy: public | restricted | private | confidential
├── relationship_context: entity_id       // which relationship it exists within
├── domains: [domain, ...]                // technical, project, research, interaction, relationship, personal, private, credentials
├── derived_from: event_hash              // provenance to source event
└── knowledge_provenance: { ... }         // if generalized, who it was learned from
```

Privacy is inferred from the structure of memory — who is involved and what the memory concerns:

```
subject=Admin + topic=personal_life -> private domain
subject=Admin + topic=runtime_architecture -> technical domain
subject=sub_user + topic=anything -> restricted (ceiling)
origin_channel=public_stream -> public (floor)
```

Admin can override inferred privacy levels.

Validated knowledge is extracted from situation model and narrative memories through corroboration, testing against conflicting evidence, and consistency checks. A fact becomes knowledge only when it survives repeated validation across independent events. Facts are held in a "hypothesis" state when contradictory evidence exists. Once a fact reaches sufficient confidence, it enters the knowledge graph.

Knowledge carries **dual access levels**: factual content (globally accessible when domain permissions allow) and contextual metadata (scoped to the originating relationship). When knowledge is accessed by the originating entity, both factual content and contextual metadata are returned. When accessed by a different entity with matching domain permissions, only the factual content is returned — the contextual metadata (who taught this, under what relationship) is stripped or anonymized. This prevents accidental leakage of contextual information while allowing useful abstractions to propagate.

The pipeline is itself a chain of projections. Each stage — situation model to narrative memories, narrative memories to validated knowledge — is a projection transforming graph state into graph state.

### 8.2 Knowledge Time Semantics

Knowledge is not final. It carries temporal semantics:

**Temporal validity** — knowledge is valid within a time window. "User prefers Rust" may be true for a project phase but not indefinitely. Knowledge entries carry `valid_from` and `valid_until` timestamps. When `valid_until` passes without corroboration, confidence decays.

**Confidence decay** — knowledge that is not corroborated over time loses confidence. The decay rate depends on the knowledge domain: technical facts decay slowly, social facts decay faster, environmental facts decay fastest. When confidence drops below the acceptance threshold, knowledge reverts to hypothesis status.

**Revision chains** — when knowledge is contradicted, the old version is not deleted. It is superseded by a new version with a causal link to the contradiction. The full revision history is preserved in the knowledge graph, following the same Git-like versioning model as all other graphs. A fact can be true at time T and false at time T+1 — both versions coexist with temporal validity windows.

### 8.2.1 Temporal Patterns

Knowledge entries can carry explicit temporal patterns derived from memory chains. When lived experience shows consistent timing — the user asks for weather every morning, checks email after lunch, or has a recurring meeting — the summarization pipeline extracts these patterns into structured temporal expressions.

**Recurring patterns** — behaviors that repeat on a schedule. Derived from memory chains where the same behavior appears at consistent times or under consistent conditions. A knowledge entry for "user checks weather daily at ~7 AM" carries a schedule expression, a confidence score, and provenance back to the memory events that established it.

**One-shot patterns** — single future events. Created when the user explicitly requests a future action ("remind me tomorrow at 3 PM") or when a time-bound knowledge entry implies a single upcoming trigger ("project deadline is Friday at 5 PM"). The intent fires once, then the pattern is removed from the active schedule.

**Pattern detection and suggestion.** The RPU assists in identifying temporal patterns from narrative memory chains during the summarization pipeline. When a consistent pattern is detected, the RPU proposes a scheduled or one-time cron entry to the user through an active channel (Web UI, Discord, CLI). The suggestion includes the pattern description, proposed schedule, and the actions it would trigger. The user approves, modifies, or rejects. Approved entries become scheduled cron jobs stored in the knowledge graph.

**Destructive/non-destructive classification.** Each tool service declares its own destructiveness level — whether it only fetches data (read-only) or changes state (write/delete/modify). This classification gates automation behavior:

- **Non-destructive cron entries** may auto-execute once the pattern's confidence score crosses a configurable threshold. The system starts by notifying the user; if the user consistently accepts, confidence rises. If the user dismisses, confidence decreases.
- **Destructive cron entries** always require explicit per-execution user approval, regardless of confidence level. The notification presents the proposed action and awaits confirmation.

**The Temporal Intent Generator** is a kernel component that manages the cron-like scheduling subsystem. It maintains an internal schedule indexed by next-fire time. When a cron entry's condition is met, it generates an intent event that flows through the same Intent → Proposal → Commit pipeline as all other intents. The generated intent carries provenance back to the knowledge entry, so the full causal chain is traceable. No RPU invocation is needed at fire time — the schedule itself is the proposal.

**The temporal intent lifecycle:**

```
Pattern Detected (RPU-assisted)
    ↓
Suggestion Presented to User
    ↓
User Approves → Cron Entry Created
    ↓
Fires Automatically (generates intent event)
    ↓
If Non-Destructive: auto-executes above confidence threshold
If Destructive: requires per-execution approval
    ↓
User Dismissal → Confidence Decay → Possible Demotion
```

**The feedback loop** closes through user interaction. When a temporal intent fires and the system notifies the user, the user's response — acceptance, denial, or modification — is recorded as an execution event. This event feeds back into the knowledge validation pipeline:

- **Acceptance** — corroborates the temporal pattern, increasing confidence
- **Denial/Dismissal** — counts as contradictory evidence; repeated dismissals reduce confidence and eventually demote the cron entry back to "suggested" status
- **Modification** — updates the temporal expression (e.g., "not now, do it at 8 AM instead") and adjusts the schedule

This creates a self-correcting automation system. Patterns that serve the user persist and strengthen; patterns that don't are pruned through natural interaction. No explicit configuration is required — the system learns when to act and when to stop from lived experience.

**Invariant clarification:** The core invariant "AI proposes intent; the kernel executes deterministically" (Section 1.4) is preserved. Temporal intent is generated by a deterministic kernel component (the temporal intent generator) from approved cron entries — not by the RPU at fire time. The RPU's role is limited to pattern detection and suggestion, which occurs during the background summarization pipeline. The cron entry itself is a user-approved schedule; its execution is a deterministic kernel operation.

### 8.3 Indexing Strategy

Each sub-graph uses different indexing strategies appropriate to its domain:

The perception graph indexes by spatial location, time window, sensor type, and derived semantic tags. This enables queries like "what did the system perceive in this room during this time period."

The execution graph indexes by chain ID, priority tier, tool type, and outcome status. This enables queries like "what chains failed in the last hour" or "which tools are most frequently used."

The memory graph indexes by semantic similarity, temporal proximity, emotional valence, and relationship context. This enables queries like "what past experiences are relevant to the current situation."

The knowledge graph indexes by semantic topic, confidence score, validation status, source provenance, and contradiction history. This enables queries like "what facts are known about topic X and how confident are we."

The summarization pipeline is what makes the two-store, seven-graph model operationally viable. Without it, queries across the full event log would be prohibitively expensive. With it, each graph operates on appropriately summarized data, and the memory and knowledge graphs emerge naturally from the pipeline at different compression levels.

The pipeline runs continuously, with summarization depth adjustable based on compute availability. In dynamic repartition mode, the offline optimization loop can re-summarize historical situation model with improved algorithms, updating the memory and knowledge graphs without touching the immutable event log.

### 8.4 Retrieval Model

Memory retrieval is the intersection of multiple filters. Only memories satisfying all constraints become part of active reasoning:

```
Candidate Memory
    ∩ Entity Policy (accessible domains, privacy levels)
    ∩ Channel Policy (allowed/blocked domains, privacy ceiling)
    ∩ Memory Domain (task-relevant domains)
    ∩ Privacy Rules (privacy level <= effective ceiling)
    ∩ Task Relevance (semantic similarity to current task)
    = Active Context
```

The retrieval pipeline operates in five steps:

1. **Entity Trust Check** — if the requesting entity is untrusted, return empty context.
2. **Channel Policy** — compute effective domains (entity ∩ channel) and effective privacy ceiling (min of entity and channel).
3. **Domain and Privacy Filter** — query memory and knowledge graphs with effective constraints.
4. **Relationship Context Filter** — prioritize memories where the entity is directly involved; apply dual-access knowledge projection (full metadata for originating relationships, stripped metadata for cross-relationship access).
5. **Task Relevance** — semantic similarity scoring, ranking, and truncation.

The output is a `ContextProjection` that becomes the context field in the RPU request.

→ See [RETRIEVAL.md](./core/RETRIEVAL.md) for the full retrieval pipeline specification.

---

## 9. Artifact Graphs

The macro graph, skill registry, and code registry are stored in the artifact version store — a content-addressed, semantically versioned, lifecycle-managed store. These are versioned entities, not event projections. They reference world-state events for provenance but are not themselves events.

**Macros are for the runtime. The code registry is for software development. They never intersect.**

Both systems share TypeScript + Effect as a common primitive vocabulary and the same Git-like, versioned storage model. This is an implementation convenience, not a conceptual unity.

### 9.1 Macro System

Macros are compiled execution subgraphs stored in the artifact version store as versioned entities. They capture _whatever repeated_ — tool call sequences, reasoning hints, plan artifacts, or any combination thereof — as deterministic conditional logic. A macro is not a reasoning engine — it executes to completion as a deterministic piece of the execution graph.

The macro does not mandate what is captured. The discovery pipeline detects the repeating subgraph in execution traces (from the world-state event store), and the macro compiles it as-is into a new artifact version. The captured content may include tool calls (always), reasoning hints (when branch points repeated), plan artifacts (when the planning-first chain repeated), or any combination.

Macro definitions live in the artifact version store. Macro execution traces are recorded in the world-state event store.

Macros use Tool Services only — they never use Application Services.

Macros are runtime-only — they execute in the TypeScript/Effect runtime and are not transpiled.

#### 9.1.1 Reasoning Hints

A Reasoning Hint is a distilled textual annotation written by the model during macro compilation. It captures the outcome of a reasoning step at a branch point, not the full chain-of-thought that produced it. Hints serve three purposes: provenance (explaining why a branch was taken), context injection (passed to the RPU as a sliding window of recent decisions), and future learning (giving macro discovery situation model context to work with).

Reasoning Hints are captured when the repeating subgraph includes branch points where reasoning occurred. If the repetition is purely sequential tool calls with no branching, the macro may have no hints.

This compression transforms expensive reasoning into reusable structure:

```
Open Reasoning (RPU with full context)
    ↓
Reasoning captured as Hint annotations on compiled macro
    ↓
Macro executes deterministically (conditional logic + tool calls)
    ↓
When new reasoning needed → kernel invokes RPU with sliding window of recent hints + tool results
    ↓
Less future reasoning required
```

→ See [MACROS.md](./core/MACROS.md#reasoning-hints) for the full treatment: hint properties, context window integration, and the discovery pipeline.

Repeated execution patterns are detected through sliding window mining over event streams, subgraph matching in trace graphs, and normalization of tool calls. The macro system operates as a reversible execution compression pipeline. Macros have two discovery sources: **pattern-derived** (discovered from general execution traces) and **skill-derived** (discovered from skill execution traces, carrying provenance links).

The process follows three steps. A macro is proposed, assisted by LLM analysis of trace patterns. It is validated through tests against historical execution data. Once validated, it is compiled into a reusable execution unit. It can be demoted if it falls out of use, if underlying primitives change, or if its source skill is updated.

Macros are compiled execution subgraphs, not behavior overrides. They are an optimization layer for execution efficiency and reliability. They never replace primitives, and they remain fully reversible.

#### 9.1.2 Macros as Compiled Commit Ranges

Frequently occurring subgraphs are identified, validated, and compiled. A macro can always be expanded back into its originating events. This guarantees that no behavior is ever hidden behind macro abstraction — the full provenance chain remains accessible.

#### 9.1.3 Macro Promotion Criteria

Not every repeated pattern should become a macro. A pattern is promoted only when it meets multiple criteria:

- **Frequency** — the pattern occurs often enough that compilation yields measurable efficiency gains
- **Success rate** — the pattern completes successfully across diverse contexts, not just in narrow conditions
- **Stability** — the underlying primitives and data shapes are unlikely to change in ways that would invalidate the macro
- **Cost savings** — the macro reduces compute time, token usage, or resource consumption compared to raw execution
- **Situation model equivalence** — the macro's behavior is functionally identical to the original pattern, not an approximation
- **Provenance validation** (skill-derived only) — the macro's behavior is equivalent to its source skill's intent, not just the observed trace

These criteria prevent the system from generating low-frequency, brittle, or marginally useful patterns that add complexity without value.

#### 9.1.4 Discovery as an Open Problem

Detecting useful reusable patterns is fundamentally difficult. The macro system's discovery mechanism is an active area of development. The criteria above provide a starting framework, but the actual mining algorithms, subgraph matching strategies, and situation model equivalence detection remain open research questions. The system is designed to evolve its discovery mechanism without changing the macro compilation, validation, or execution model.

Skill-derived macro discovery adds a structured seed to this problem: when skills are installed, they provide known-good execution traces with clear intent, making the equivalence check more tractable. However, parameter extraction from skill instruction templates and the quality of skill-to-macro transformation remain open questions. Skills are optional — pattern-derived macros are discovered from general execution regardless of skill presence.

#### 9.1.5 Skill-Derived Macros

When skills are installed and executed, macros discovered from their execution traces carry provenance metadata linking them to their source skill and version:

```json
{
  "provenance": {
    "source_type": "skill",
    "source_id": "skill_weather_check",
    "source_version": "1.2.0",
    "discovered_at": "2024-03-15T08:00:00Z",
    "execution_count": 47
  }
}
```

**Routing logic.** When a request matches a skill, the kernel checks for a derived macro with matching provenance version. If one exists and is promoted with high confidence, the macro executes (fast path, deterministic). Otherwise, the skill executes through the RPU (normal reasoning path). Both paths produce execution traces that feed the discovery pipeline.

**Skill version updates.** When a skill is updated, all derived macros are flagged for re-validation. The existing demotion mechanics handle this: derived macros are demoted (status: `source_updated`) until re-validated against the new skill version. Knowledge entries derived from the old skill version enter accelerated confidence decay. If the new skill produces the same behavioral patterns, new knowledge entries are created with provenance to the new version.

→ See [MACROS.md](./core/MACROS.md#skill-derived-macros) for the full treatment: provenance schema, routing decision tree, version-mismatch handling.

#### 9.1.6 Hierarchical Macros

Macros can reference other macros as child subgraph components. Since the execution model is event-driven (not call-driven) and the world-state graph is a DAG by construction, hierarchical macros inherit the acyclic property — circular references are impossible.

**Composition model.** A hierarchical macro is not a runtime call stack. It is a logical grouping of subgraph references. At execution time, child macros are inlined into the parent's execution sequence — sequential or parallel event execution, not function calls. The hierarchy is a design-time abstraction for discovery partitioning and maintenance, not a runtime structure.

```
Macro: morning_routine
    ├── Macro: weather_check (child subgraph reference)
    ├── Macro: calendar_briefing (child subgraph reference)
    └── Macro: news_summary (child subgraph reference)
```

**Discovery paths.** Bottom-up: leaf macros (tool-call patterns) are discovered first due to higher frequency; later, the system detects that certain leaf macros are invoked together repeatedly and proposes a parent macro. Top-down: a flat macro is discovered first; during normalization, the system identifies reusable sub-patterns and extracts them as child macros (analogous to "extract function" refactoring).

**Parameter passing.** Parent macros pass parameters to children through binding: literal values, parent parameter references (`{parent.location}`), user context references (`{user_home_location}`).

**Error propagation.** When a child macro fails, the parent's strategy is captured in a Reasoning Hint: abort (stop entire macro), skip (continue without child output), fallback (use default value), or escalate (invoke RPU for reasoning).

**No depth limits.** Macros are always expandable to their full event ranges — like folders in a tree view, they can be recursively expanded to reveal the complete execution history. Tracing is a UI concern, not an architectural constraint. The provenance chain from parent macro through child macros to leaf tool calls is always preserved.

**Demotion propagation.** When a child macro is demoted, the parent macro may be demoted, or may fall back to direct tool calls for that child's subgraph. The decision depends on whether the child's functionality is critical to the parent's intent.

→ See [MACROS.md](./core/MACROS.md#hierarchical-macros) for the full treatment: composition schema, discovery algorithms, parameter binding, error strategies.

#### 9.1.7 Captured Content and Plan Artifacts

A macro captures whatever the discovery pipeline detects as the repeating subgraph. The captured content exists on a spectrum:

- **Tool calls only** — the simplest macro: sequential or conditional tool calls with no branching reasoning and no plan artifact.
- **Tool calls + reasoning hints** — the common case: branch points where reasoning occurred are captured as hints alongside the tool calls.
- **Tool calls + reasoning hints + plan artifact** — the full case: the entire planning-first execution chain repeated, so the plan artifact generated by the RPU is captured alongside everything else.

The macro does not mandate what is captured. It compiles the detected pattern as-is.

**Plan artifact capture.** When the repeating subgraph includes a planning-first execution chain, the plan artifact generated by the RPU is captured as part of the macro. The captured plan serves three purposes when present:

1. **Observability** — the user sees the macro's intent at the plan level, not just the tool-call level.
2. **Escalation context** — when the macro needs RPU reasoning (novel situation, child macro failure), the captured plan provides rich intent context for the RPU to reason about.
3. **Intent trace** — preserves the original reasoning intent even in deterministic execution.

Plan artifact capture is contingent on the detected pattern. If the repetition is at the tool-call level, the macro captures tool calls (and hints if present) without a plan artifact. If the repetition includes the full planning-first chain, the plan artifact is captured alongside everything else.

Reasoning Hints and captured plan artifacts are complementary: hints capture branch-level reasoning outcomes ("why this decision"), while the plan artifact captures the chain-level intent structure ("what we're doing").

→ See [MACROS.md](./core/MACROS.md#captured-content) for the full treatment: content spectrum, plan artifact structure, and contingent capture mechanics.

### 9.2 Code Registry

The code registry is **not part of the runtime's execution path**. It is a repository of tested, versioned code components stored in the artifact version store, that the model, agents, and users draw from when writing software together. When the system helps a user build an application, it pulls from the registry instead of generating code from scratch.

The code registry and the runtime's orchestration layer share TypeScript + Effect as a common primitive vocabulary for transpilation, but they never intersect. Macros compose Tool Services at runtime. Code components compose Application Services during software development.

**The problem.** Most application code consists of common patterns: data transformation, validation, formatting, state management, error handling. These are repeatedly written, rarely tested thoroughly, and pulled from external packages that may carry security risks or unnecessary dependencies.

**The solution.** Build an npm-like repository where code components are written once, tested thoroughly, and versioned. The model pulls from the registry when writing new code instead of reinventing logic. Every component has a full history — who wrote it, what tests it passes, what depends on it, how it evolved. Quality improves through reuse: frequently used components get more testing, edge cases get discovered, they get refined. Safety comes from versioning: you know exactly what code is included in generated programs, it's been validated, and you can roll back.

Code components are Effect-style function definitions that compose Application Services through semantic orchestration primitives. Code components use Application Services only — they never use Tool Services.

Code components support transpilation across language runtimes. TypeScript + Effect transpiles nearly 1:1 to Go due to structural similarity in their concurrency models — goroutines, contexts, and channels map closely to Effect primitives. C++ requires a runtime layer to support the same semantic primitives, reserved for environments like Unreal Engine that expect native code. Native code modules can be exposed as Application Services, enabling cross-language composition.

The code registry applies the same Git-like, event-sourced model used for world-state and macros to the domain of versioned code components. It is a homelab-hosted repository where every piece of code — written by the model, by autonomous agents, or by the user — is tested, versioned, and stored as a reusable, content-addressed artifact.

#### 9.2.1 Relationship to Macros and Skills

Macros, code components, and skills are three applications of the same underlying pattern: repeated behavior is captured, validated, and made reusable. The distinction is in discovery, validation, and purpose. All three are stored as versioned entities in the artifact version store.

| Runtime                           | Software                   | Seed               |
| --------------------------------- | -------------------------- | ------------------ |
| Macro Graph                       | Code Registry              | Skill Registry     |
| Execution compression             | Implementation compression | Behavior templates |
| Passive discovery + skill-derived | Active authoring           | Pre-authored       |
| Historical validation             | Test validation            | Author-defined     |

Macros are passively discovered through trace mining and validated against historical execution data. They compress the runtime's own execution patterns.

Code components are actively written and must pass a mandatory test pipeline before adoption. They compress implementation patterns used in software the system helps users build.

Skills are pre-authored behavior definitions that seed the system with competent behavior from day one. They are executed through the RPU and generate execution traces that feed the macro discovery pipeline. Skills are the input; macros are the optimized output.

#### 9.2.2 Mandatory Test Pipeline

Unlike macros, which are passively discovered and validated against historical data, code components must actively pass a test pipeline before they are adopted into the registry. Every new version of a code component must:

- Pass its full test suite
- Maintain or improve coverage thresholds
- Not introduce regressions in dependent components
- Be compatible with declared dependencies

The test pipeline runs in the homelab's trusted environment. Only components that pass all checks are published to the registry. Failed versions are recorded in the event log but never become available for use.

#### 9.2.3 Homelab as Trusted Source

The code registry lives on the homelab as the trusted source of truth. Edge nodes pull from it when needed — initially rarely, but as the system grows, edge devices including small laptop setups and the PEN may pull tested components to use when writing software locally.

The registry is versioned and content-addressed. Edge nodes pin to specific versions and can verify integrity through cryptographic hashes. No untested or unversioned code is ever distributed.

#### 9.2.4 Goal: In-House Logic, Minimal External Dependencies

Core infrastructure packages remain external: frameworks, bundlers, compilers, database drivers. These are infrastructure you don't reimplement.

But the logic that glues them together becomes internal: parsers, validators, formatters, state machines, utility functions, domain-specific logic. Over time, the system accumulates a tested, versioned library of in-house components that the model draws from when writing software, replacing external dependencies for small logic.

→ See [REGISTRY.md](./core/REGISTRY.md#dependency-taxonomy) for the full dependency taxonomy and transpilation details.
→ See [MACROS.md](./core/MACROS.md#discovery-mechanisms) for mining algorithms, validation test design, and demotion mechanics.

### 9.3 Skill Registry

The Skill Registry is a repository of pre-authored behavior definitions in the existing harness format, stored in the artifact version store as versioned entities. Skills are instruction files (prompts, system instructions, tool definitions) that users _may_ install to give the system competent behavior in specific domains. Skills are not a new format — they are the same skill files used by existing AI harnesses, making migration seamless.

**The problem (optional).** Without pre-authored behaviors, the system starts by reasoning from scratch for every task until it accumulates enough experience to discover patterns. Skills are an optional acceleration — they provide competent behavior from the start, but the system works without them.

**The solution.** Skills provide optional seed behaviors. A skill defines what the system should do in a given domain — which tools to use, what prompts to follow, how to handle edge cases. The system executes skills through the RPU (same as any other reasoning), but the execution traces enter the event graph. The compression pipeline then converts skill-driven reasoning into derived macros (deterministic, personalized) and derived knowledge (validated facts with provenance). Without skills, the same pipeline operates on general execution traces.

Skills are content-addressed and versioned with semantic tags (`major.minor.patch`). The content hash is the canonical identifier; semantic tags are mutable pointers for human readability. Users may install zero, one, or many skills. Skills can be installed at any time — during initial setup, or later as the user discovers new needs. The system may also suggest skills based on observed behavior patterns that match available skill capabilities.

#### 9.3.1 Skill Format

Skills use the existing harness format — no new format to learn, no migration required. A skill consists of:

- **Instruction file** — system prompt, task description, behavioral guidelines
- **Tool references** — declarations of which tools the skill uses
- **Parameter definitions** (optional) — configurable values the user can override
- **Version metadata** — semantic version, author, description

The system does not parse or interpret skill instructions directly. Skills are executed through the RPU like any other reasoning task. The skill's instructions become part of the RPU request context. This means the system can optimize any skill regardless of its internal structure — the optimization happens at the execution trace level, not the instruction level.

#### 9.3.2 Open Ecosystem

Skills come from any source: the system developer, the user themselves, third-party authors, community repositories. The system does not curate or approve skills. It is the user's responsibility to install skills they trust.

**Security audit (advisory, Phase 4).** The system includes a Skill Security Audit activity that analyzes installed skills for known risk patterns: prompt injection vectors, dangerous tool access, data exfiltration patterns, privilege escalation attempts. The audit can be manually triggered by the user when viewing a skill, or run as a background task during offline periods. Audit findings are recorded as events in the execution graph and surfaced to the user. The audit is advisory — it raises events but does not block skill execution. The user decides whether to act on audit findings. This feature serves as the first implementation of background/offline work structure in the system.

#### 9.3.3 Skill Execution and Compression

Skills are not executed differently from other behavior. When a request matches a skill:

1. The kernel constructs an RPU request with the skill's instructions, tool references, and current context.
2. The RPU executes the skill (full reasoning path).
3. The execution trace is recorded in the execution graph.

**Skill execution is available from Phase 4.** The system executes skills through the RPU with full reasoning, using memory and knowledge for context. This makes the harness functionally complete — the system can handle skill-covered tasks competently from day one of Phase 4.

**Skill-to-macro compression is a Phase 5 optimization.** The offline discovery pipeline (Phase 5) analyzes accumulated skill execution traces for repeated patterns. When a pattern is detected, a skill-derived macro is proposed with provenance metadata, validated, and promoted. The macro then becomes the fast path for future executions.

The skill remains available as the reasoning path at all times. The derived macro becomes the fast path when promoted. Both coexist; the kernel routes based on provenance version match and confidence.

#### 9.3.4 Relationship to Macros and Knowledge

| Runtime                           | Software                   | Seed               |
| --------------------------------- | -------------------------- | ------------------ |
| Macro Graph                       | Code Registry              | Skill Registry     |
| Execution compression             | Implementation compression | Behavior templates |
| Passive discovery + skill-derived | Active authoring           | Pre-authored       |
| Historical validation             | Test validation            | Author-defined     |

Skills are the seed. Macros are the compressed, personalized derivative. Knowledge is the factual extraction. All three are linked by provenance.

→ See [MACROS.md](./core/MACROS.md#skill-derived-macros) for the skill-to-macro transformation pipeline.

---

## 10. Semantic Orchestration Layer

The system defines a minimal set of Effect-like orchestration primitives that are semantically equivalent across all runtime backends. These primitives define what a program means, independent of any programming language.

### 10.1 Core Primitives

- `run` — execute a task
- `fork` — spawn a concurrent task
- `parallel` — run multiple tasks concurrently
- `race` — run multiple tasks, take the first result
- `scope` — manage task lifecycle and cancellation
- `acquire` / `release` — resource management with guaranteed cleanup
- `fail` / `recover` / `mapError` — error handling and transformation
- `stream` / `pipe` / `sink` — streaming data flow
- `provide` / `layer` / `context` — dependency injection and service composition

These are not a domain-specific language. They are a portable execution meaning layer.

### 10.2 Tool Services

Tool Services are exposed to macros. Individual tool calls are exposed as service methods. Macros compose these services through semantic orchestration primitives to execute automations — combining tool calls with conditional logic to capture reasoning and tool usage patterns.

Tool Services are runtime-only. They are not transpilable and exist only within the TypeScript/Effect runtime.

### 10.3 Application Services

Application Services are exposed to code components. These are application-level capabilities — databases, APIs, storage, and native code modules. Code components compose these services through semantic orchestration primitives.

Application Services support transpilation across language runtimes. TypeScript + Effect transpiles nearly 1:1 to Go due to structural similarity in their concurrency models. C++ requires a runtime layer to support the same semantic primitives, reserved for environments like Unreal Engine that expect native code. Native code modules can be exposed as Application Services, enabling cross-language composition.

Code components use Application Services only — they never use Tool Services.

### 10.4 Consumers

These primitives are consumed by two derived graph types (section 9):

- **Macros** use Tool Services only, runtime-only, no transpilation
- **Code components** use Application Services only, with transpilation support

The two ecosystems never intersect.

→ See [ORCHESTRATION.md](./core/ORCHESTRATION.md#core-primitives--semantic-specifications) for per-primitive semantic descriptions and concrete composition examples.

---

## 11. Running Example: A Request Through the Full Pipeline

To make the architecture concrete, consider a single user request: _"What's the weather today?"_

**1. Signal** — The user speaks the command. Audio is captured by the microphone on the edge node.

**2. Perception Processing** — Speech-to-text converts the audio stream into structured perception: `{ type: "perception", modality: "audio", transcript: "what's the weather today", confidence: 0.96 }`. This event is recorded in the perception graph.

**3. Situation Model Generation** — The RPU receives the perception along with context (user location from perception graph, time of day, personality state). It produces a situation model: `{ type: "situation_model", interpretation: "user wants current weather forecast for their location", confidence: 0.92 }`.

**4. Intent Estimation** — The situation model converges into an intent: `{ type: "intent", source: "user", desire: "get weather forecast", confidence: 0.92 }`.

**5. Proposal** — The kernel translates intent into a proposal: call the weather service, format the result, send it to the user's active channel. Resource estimate: 500 tokens, 2 seconds.

**6. Commit** — The scheduler validates the proposal (interactive priority, within resource quota) and commits it as an execution chain.

**7. Execution** — The chain executes: acquire weather service → fetch forecast → format response → send to channel. If a macro exists for "weather check + notification," it executes deterministically with a Reasoning Hint explaining the branch taken.

**8. Memory** — The execution outcome is summarized into a narrative memory: "User asked for weather on Jan 1, provided forecast for Boston, user acknowledged."

**9. Knowledge** — If this pattern repeats (user asks for weather every morning), the system may extract knowledge: "User checks weather daily, prefers morning forecast, location is Boston." This knowledge carries a validity window, confidence score, and a temporal pattern derived from the memory chain: `{ schedule: "daily at 07:00", type: "recurring", confidence: 0.88 }`.

**10. Temporal Intent** — Over time, the pattern "user asks for weather every morning" is detected by the summarization pipeline. The RPU proposes a daily cron entry at 7 AM. The user approves. The temporal intent generator stores the cron entry. When 7 AM arrives, it produces an intent event: `{ type: "intent", source: "temporal", desire: "get weather forecast", triggered_by: "knowledge_xyz", schedule_type: "recurring" }`. This flows through the same proposal → commit pipeline. Since fetching weather is a non-destructive tool service, the entry auto-executes above the confidence threshold. The user receives a notification; if they dismiss it, the pattern's confidence decreases.

**11. Compression** — If a macro exists for the weather check pattern, the temporal intent triggers the macro instead of full reasoning. The macro executes deterministically with a Reasoning Hint: "User typically wants weather at 7 AM, proactively offering."

Each step is an immutable event in the world-state graph. The full provenance chain is traceable. The system can answer: why was the weather fetched, what perception triggered it, what situation model was generated, what chain executed it, and what knowledge was derived.

### 11.1 Running Example with Skills

The same scenario, viewed through two lenses:

**With skills (optional acceleration).** The user has installed `skill_morning_briefing` (v1.0), which includes weather check, calendar summary, and news digest. The request "What's the weather today?" matches the skill. The kernel constructs an RPU request with the skill's instructions, enriched by memory and knowledge context. The RPU executes the full reasoning path: generates a plan, fetches weather, formats response, sends to channel. The execution trace — including the plan artifact — is recorded. Tokens consumed: ~2,000. The harness is functionally complete.

**Phase 5 (pattern detection).** The skill has executed 15+ times. The offline discovery pipeline detects a repeated subgraph: the full planning-first chain including the plan artifact, weather fetch, format, and send. A skill-derived macro is proposed: `macro_weather_check_v1`, with provenance `{ source: "skill_morning_briefing", version: "1.0" }`. The captured plan artifact is preserved as part of the macro.

**Phase 5 (macro validation).** The proposed macro is replayed against historical traces. It produces semantically equivalent results with 80% fewer tokens. It is promoted.

**Phase 5 (fast path active).** The same request now routes to `macro_weather_check_v1` (provenance match confirmed). The macro executes deterministically. The captured plan artifact is available for observability — the user sees "Weather Check: fetch forecast, format, send" even though no reasoning is happening. Tokens consumed: ~400. The skill remains available as fallback.

**Without skills (organic discovery).** The user has no skills installed. The request "What's the weather today?" triggers general RPU reasoning: situation model generation, plan creation, tool execution. The execution trace is recorded. Tokens consumed: ~2,000. Over time, the same weather-check pattern repeats organically. The discovery pipeline detects the repeated subgraph and proposes a pattern-derived macro. The macro is validated and promoted. The result is the same — deterministic execution at ~400 tokens — but the path to get there is slower because the system had to discover the pattern on its own.

This progression demonstrates the core thesis: the same behavior that required 2,000 tokens of reasoning in Phase 4 requires 400 tokens of deterministic execution in Phase 5 — with or without skills. Skills accelerate the path; they don't change the destination.

---

## 12. Edge Architecture

The edge architecture is an experimental extension of the core runtime. It explores how far the compression thesis can be pushed when perception becomes mobile and continuous. The runtime's core thesis does not depend on the PEN existing.

→ See [EDGE_ARCHITECTURE.md](./core/EDGE_ARCHITECTURE.md#hardware-topology) for the full specification: hardware topology, power and thermal constraints, network bandwidth requirements, local model specifications, and graceful degradation details.

---

## 13. Multimodal Cognitive Interface

The multimodal interface is an experimental extension. The system functions with voice and text input alone — gaze, EMG, and EEG are optional modalities that may improve interaction quality but are not required for the core thesis.

→ See [MULTIMODAL_INTERFACE.md](./core/MULTIMODAL_INTERFACE.md#sensor-fusion-algorithm-overview) for the full specification: sensor fusion algorithm, intent fusion threshold justification, modality conflict resolution, and calibration procedure.

---

## 14. Offline Optimization

The offline optimization loop runs separately from the real-time runtime but shares the same compute infrastructure.

→ See [OFFLINE_OPTIMIZATION.md](./core/OFFLINE_OPTIMIZATION.md#dynamic-repartition-mode) for the full specification: dynamic repartition mode, offline activities, and reserved capacity guarantees.

---

## 15. Security and Privacy Considerations

The system spans edge devices, home infrastructure, and potentially cloud providers. Data boundaries, biometric handling, network security, and cryptographic integrity are defined here.

→ See [SECURITY.md](./core/SECURITY.md) for the full specification: all ten security subsections from data boundaries to observability and audit.

---

## 16. Evaluation Methodology

The core thesis — that accumulated structure can substitute for repeated inference — is falsifiable.

→ See [EVALUATION.md](./core/EVALUATION.md#primary-metric-reasoning-cost-per-task) for the full specification: primary metrics, external baseline comparison, ablation study design, statistical power analysis, task taxonomy, and quality standard definitions.

---

## 17. Known Gaps and Future Work

Several areas are acknowledged as incomplete or requiring further research.

→ See [KNOWN_GAPS.md](./core/KNOWN_GAPS.md#macro-discovery-section-91) for the full specification: all known gaps including macro discovery, knowledge extraction, cost model, failure recovery, distributed conflict resolution, and additional gaps identified during documentation review.

---

## 18. Architecture Flow Diagrams

→ See [ARCHITECTURE_DIAGRAMS.md](./core/ARCHITECTURE_DIAGRAMS.md#full-transformation-flow) for all system flow diagrams: transformation flow, unified system model, intent pipeline, summarization pipeline, macro lifecycle, cognitive cache hierarchy, and edge-cloud separation.
