# Distributed Cognitive Runtime

**Technical Concept Document**

---

## Abstract

This document describes B.Y.T.E. (Behavior Yielding Through Evolution), a distributed, event-sourced cognitive runtime architecture that unifies perception, reasoning, and action through a Git-like world-state graph. The system tests a single hypothesis: **can accumulated structure substitute for repeated inference?** Instead of improving AI agents through larger models or more context, the architecture converts past reasoning into reusable structure — macros and knowledge graphs — so that the minimum reasoning required to achieve a task decreases over time. A companion code registry provides tested, versioned components for software development tasks. The core thesis is falsifiable: if Phase 4 (Memory + Knowledge) does not demonstrate a measurable reduction in reasoning cost for equivalent outcomes, the project fails. This document defines the complete target architecture. Implementation has not yet begun.

---

## Related Documents

### Core Subsystems

- [GRAPH.md](core/GRAPH.md) — World-state graph model (diagrams, design heuristics, query complexity)
- [RPU.md](core/RPU.md) — Reasoning Processing Unit (contract, error handling, model versioning)
- [ORCHESTRATION.md](core/ORCHESTRATION.md) — Semantic orchestration primitives (semantics, composition laws)
- [MACROS.md](core/MACROS.md) — Macro system (mining algorithms, validation, demotion)
- [REGISTRY.md](core/REGISTRY.md) — Code registry (versioning, dependency resolution, transpilation)

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
Reasoning → Execution → History → Compression → Structure → Less future reasoning
```

Every layer in this document exists to support that loop. The event graph stores it. The scheduler executes it. The memory system compresses it. The macro system formalizes it.

**Non-monotonicity of structure accumulation.** The compression thesis assumes that accumulated structure grows over time, but the system includes mechanisms that reduce structure: knowledge decay (Section 8.2), macro demotion (Section 9.1.4), and revision chains (Section 8.2). This means structure accumulation is not monotonic — some structure is lost as it becomes obsolete or invalid.

The net effect is captured by the concept of **effective structure**: the total accumulated structure minus decayed, demoted, or superseded structure. The thesis claims that effective structure grows over time, even though gross structure (total ever accumulated) may exceed net structure (currently valid). The evaluation methodology (Section 16) measures reasoning cost reduction, which implicitly measures effective structure growth — if reasoning cost decreases, effective structure is growing.

The decay mechanisms are intentional: they prevent the system from accumulating stale or incorrect structure. A system that never forgets becomes burdened by outdated knowledge. The decay rates (technical facts: slow, social facts: faster, environmental facts: fastest) are designed to match the expected volatility of each domain.

### 1.4 Core Invariants

These principles govern every design decision:

- **AI proposes intent; the kernel executes deterministically.** The LLM never runs the system. It proposes structured intent into the execution kernel, which validates, schedules, and executes it using deterministic primitives.
- **Everything is an event.** There is no distinction between input and output. Sensor readings, tool calls, reasoning steps, execution chains, and environmental state are all events in the same structure.
- **The world-state is an immutable event graph, not a mutable database.** Events are append-only, content-addressed, and cryptographically verifiable. Current state is always a projection of history.
- **Execution chains are persistent, prioritized, and interruptible.** Chains survive waiting, suspension, and resumption. They are governed by priority levels and resource quotas.
- **Macros are compiled execution subgraphs, not behavior overrides.** Repeated execution patterns are discovered, validated, and compiled into reusable units. They never replace primitives.
- **Edge-cloud separation of cognition.** Real-time perception and immediate interaction logic run on edge devices. Heavy reasoning, long-term memory, and planning run on the homelab.
- **AI is an interpretation layer, not a control system.** Infrastructure and deterministic logic never depend on AI reasoning.
- **Reasoning is externalized into accumulated structure.** The objective is not to maximize model intelligence but to minimize the amount of reasoning required to produce intelligent behavior.
- **Graceful degradation is mandatory.** The system must remain functional and safe when disconnected from the homelab, when network links fail, or when sensor inputs become unreliable.

### 1.5 Implementation Note: Six Graphs, One Store

The six logical graphs (perception, execution, memory, knowledge, macro, registry) are a conceptual model for reasoning about the system. In practice, they are implemented as a single event store with multiple projection layers. The separation keeps queries simple and indexing efficient, but does not require six separate databases.

```
Single Event Store (append-only, content-addressed)
    │
    ├── Perception projection
    ├── Execution projection
    ├── Memory projection
    ├── Knowledge projection
    ├── Macro projection
    └── Registry projection
```

Each projection maintains its own indexing strategy and retention policies. Cross-graph references work like git submodules — content-addressed hashes link components across domains while keeping queries simple.

→ See [GRAPH.md](./core/GRAPH.md#one-line-architecture-summary) for visual diagrams and design review heuristics.

### 1.6 Cold-Start Problem

The system improves over time through accumulated structure, but the initial period — when there is no memory, no knowledge, and no macros — requires specific design attention.

**Phase 1 experience (no structure):** The system operates as a structured but uncompressed runtime. Every task requires full reasoning. The RPU is invoked for all interpretation, planning, and summarization. This is functionally equivalent to a well-engineered prompt-chain agent with structured contracts — better than ad-hoc agents, but not yet differentiated.

**Bootstrapping process:**

1. **Day 1:** The system has no personality, no knowledge, no memories. It uses default personality parameters and relies entirely on real-time perception and RPU reasoning.
2. **Week 1:** Narrative memories begin accumulating. The system starts recognizing patterns in user behavior but has not yet validated them as knowledge.
3. **Month 1:** Validated knowledge begins entering the knowledge graph. Temporal patterns emerge from memory chains. The first macros are proposed from repeated execution traces.
4. **Month 3+:** The compression loop is operational. Macros handle frequent tasks. Knowledge eliminates re-derivation. The system's reasoning cost per task begins to measurably decrease.

**Cold-start mitigation strategies:**

- **Default personality parameters** provide reasonable behavior from day one. The personality evolves as experience accumulates.
- **Pre-built knowledge templates** (optional) can bootstrap common knowledge domains (user preferences, common facts) to accelerate early learning.
- **User onboarding** can explicitly teach the system key preferences and patterns, jump-starting the knowledge graph.
- **The evaluation baseline (Phase 2) is measured during the cold-start period.** This ensures that the thesis is tested from the worst-case starting point, not from a pre-loaded state.

The cold-start period is not a flaw — it is the natural consequence of a system that learns from experience. The evaluation methodology (Section 16) measures improvement from this baseline, not from an idealized pre-loaded state.

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
World-State Graph (canonical event IR)
    │
    ├── Cognitive Kernel (scheduling, chain lifecycle, resource arbitration)
    ├── Perception Nodes (sensor processing, structured perception)
    └── Interaction Layer (channels, multimodal interface)
            │
            ▼
    Execution + Trace IR Layer
            │
            ▼
    Macro Optimization (reversible compilation)
            │
            ▼
    Language Adapters (TS / Go / C++ etc.)
```

### 3.2 Build Phases and Document Mapping

| Phase | Component                 | Document Sections | Core Documents                                | Status          |
| ----- | ------------------------- | ----------------- | --------------------------------------------- | --------------- |
| 1     | Kernel + Execution Graph  | Sections 4, 5     | GRAPH.md                                      | Design complete |
| 2     | RPU + Orchestration       | Sections 6, 10    | RPU.md, ORCHESTRATION.md                      | Design complete |
| 3     | Signal-to-Intent Pipeline | Section 7         | —                                             | Design complete |
| 4     | Memory + Knowledge        | Section 8         | —                                             | Design complete |
| 5     | Code Registry + Macros    | Section 9         | MACROS.md, REGISTRY.md                        | Research phase  |
| 6     | Edge + Multimodal         | Sections 12, 13   | EDGE_ARCHITECTURE.md, MULTIMODAL_INTERFACE.md | Conceptual      |

Running example: Section 11. Evaluation: [EVALUATION.md](core/EVALUATION.md).

Phases 1–4 test the core thesis. Phases 5–6 are experimental extensions.

---

## 4. World-State Graph Architecture

The world-state graph is the single source of truth for the entire system. It is an event graph where every piece of system activity is recorded as a structured event with normalized arguments, timestamps, and dependency edges.

### 4.1 Sub-Graph Architecture

Conceptually the world-state is unified. Operationally it is split into six logical graphs to keep queries simple, indexing efficient, and maintenance manageable. Each graph follows the same Git-like model — immutable events, content-addressed, DAG-structured, state-as-projection — but handles a different domain.

**Perception Graph** — structured perception from signal processing systems, environmental facts, user state, biometrics. Answers "what is happening in the world." Contains the outputs of object detection, speech-to-text, attention tracking, movement analysis, and cognitive state processing. These are the first entries in the graph — raw sensor streams never enter.

**Execution Graph** — chains, tasks, scheduling decisions, tool calls, RPU invocations. Answers "what is the system doing." Contains execution chain lifecycle, scheduler decisions, tool call records, RPU request/response pairs, and state transitions.

**Memory Graph** — episodic experiences, emotional context, narrative memories, personality evolution, relationship state. Answers "what has been experienced." Retrieved by similarity, temporal proximity, and emotional valence. Evolves as new experiences accumulate and context shifts.

**Knowledge Graph** — validated facts, semantic knowledge, tested relationships, established patterns. Answers "what is known to be true." Retrieved by semantic query. Facts must pass a validation pipeline before becoming knowledge. Knowledge carries temporal validity, confidence decay, and revision chains — it is durable but not final. Also serves as the source for temporal intent generation when knowledge carries explicit temporal patterns.

**Macro Graph** — compiled execution subgraphs. Discovery is passive through sliding window mining. Validation tests against historical traces. Answers "what patterns repeat and whether they are valid."

**Code Registry Graph** — versioned, tested code components that the model and agents draw from when writing software with users. Not executed by the runtime; used to compose new programs. Components are actively authored by the model, agents, or the user. A mandatory test pipeline validates before adoption. Answers "what code exists, what version is current, and whether it passes tests."

Each graph maintains its own projection layer for queries, its own indexing strategy, and its own retention policies. The event log remains the source of truth for all six.

### 4.2 Cross-References Between Sub-Graphs

Graphs reference each other through content-addressed hashes. A perception (dog detected) can be referenced by execution (chain triggered object interaction). An execution outcome (task completed) can be referenced by memory (experience indexed). A memory lookup (recall past interaction) can be referenced by execution (context injected into RPU request). A validated fact (Bob prefers Rust) can be referenced by execution (code generation chose Rust for Bob's project). Cross-graph references work like git submodules — content-addressed hashes link components across domains while keeping queries simple.

### 4.3 Events as Immutable Commits

Every event is immutable. Events reference their causal parents, are cryptographically hashed and optionally signed, and the history is append-only. This creates a tamper-evident record of all system activity.

### 4.4 State as Projection

Current state is derived from replaying events. State is a materialized view, not the source of truth. Checkpoints accelerate reconstruction but do not replace history. The system can always rebuild current world-state from the full event log.

### 4.5 Projections and Channels

The runtime distinguishes between two categories of transformation over the event graph:

**Projections** transform graph state into graph state. They are internal runtime transformations that produce durable, queryable, composable knowledge structures. A projection is a pure function over an event stream:

```
Graph → Projection → Graph
```

Examples:

```
Perception → Perception Graph
Perception Graph → Situation Model
Situation Model + User Signals → Intent
Intent → Execution Graph
Execution Graph → Memory Graph
Memory Graph → Knowledge Graph
Execution Graph → Macro Graph
```

Projections optimize for correctness, queryability, persistence, and composition. They can be chained — the output of one projection becomes the input of another. The summarization pipeline (section 8) is a chain of projections: situation model into narrative memories, into validated knowledge.

**Channels** transform graph state into external representation. They are bidirectional boundaries between the runtime and the outside world. A channel consumes projected state and translates it into a consumer-facing interface, while also translating external inputs back into events:

```
Graph → Channel → External Surface
External Surface → Channel → Event
```

Examples:

```
Execution Graph → Web UI Channel
Execution Graph → Discord Channel
Execution Graph → CLI Channel
Memory Graph → Search API Channel
Execution Graph → Telemetry Channel
Perception Graph → Streaming Channel
```

Channels optimize for rendering, filtering, sorting, formatting, and aggregation. They are leaves in the transformation pipeline — nothing projects further from a channel. Channels are replaceable and independent; the same projection graph can serve any number of channels without conceptual changes.

The distinction matters because projections build runtime knowledge while channels consume it. Projections are durable; channels come and go. The browser, Discord, voice interfaces, and the wearable edge node are all channels attached to the same projection graph.

**Decision rules:** If a transformation doesn't produce durable runtime knowledge, it's a channel, not a projection. If a channel tries to produce runtime state, it should emit events instead. If a new subsystem doesn't fit naturally into this model, it's introducing a second source of truth.

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

Validated knowledge is extracted from situation model and narrative memories through corroboration, testing against conflicting evidence, and consistency checks. A fact becomes knowledge only when it survives repeated validation across independent events. Facts are held in a "hypothesis" state when contradictory evidence exists. Once a fact reaches sufficient confidence, it enters the knowledge graph.

The pipeline is itself a chain of projections. Each stage — situation model to narrative memories, narrative memories to validated knowledge — is a projection transforming graph state into graph state.

### 8.2 Knowledge Time Semantics

Knowledge is not final. It carries temporal semantics:

**Temporal validity** — knowledge is valid within a time window. "User prefers Rust" may be true for a project phase but not indefinitely. Knowledge entries carry `valid_from` and `valid_until` timestamps. When `valid_until` passes without corroboration, confidence decays.

**Confidence decay** — knowledge that is not corroborated over time loses confidence. The decay rate depends on the knowledge domain: technical facts decay slowly, social facts decay faster, environmental facts decay fastest. When confidence drops below the acceptance threshold, knowledge reverts to hypothesis status.

**Revision chains** — when knowledge is contradicted, the old version is not deleted. It is superseded by a new version with a causal link to the contradiction. The full revision history is preserved in the knowledge graph, following the same Git-like versioning model as all other graphs. A fact can be true at time T and false at time T+1 — both versions coexist with temporal validity windows.

### 8.2.1 Temporal Patterns

Knowledge entries can carry explicit temporal patterns derived from memory chains. When lived experience shows consistent timing — the user asks for weather every morning, checks email after lunch, or has a recurring meeting — the summarization pipeline extracts these patterns into structured temporal expressions. The temporal intent generator monitors the knowledge graph for active patterns and produces intent when conditions are met.

**Recurring patterns** — behaviors that repeat on a schedule. Derived from memory chains where the same behavior appears at consistent times or under consistent conditions. A knowledge entry for "user checks weather daily at ~7 AM" carries a schedule expression, a confidence score, and provenance back to the memory events that established it. When the schedule fires, the temporal intent generator produces an intent: `{ source: "temporal", desire: "get weather forecast", triggered_by: "knowledge_xyz", schedule_type: "recurring" }`.

**One-shot patterns** — single future events. Created when the user explicitly requests a future action ("remind me tomorrow at 3 PM") or when a time-bound knowledge entry implies a single upcoming trigger ("project deadline is Friday at 5 PM"). The intent fires once, then the pattern is removed from the active schedule.

**The Temporal Intent Generator** is a kernel component that monitors the knowledge graph for active temporal patterns. It maintains an internal schedule indexed by next-fire time. When a pattern's condition is met, it generates an intent that flows through the same Intent → Proposal → Commit pipeline as all other intents. The generated intent carries provenance back to the knowledge entry, so the full causal chain is traceable.

**The feedback loop** closes through user interaction. When a temporal intent fires and the system offers help, the user's response — acceptance, denial, or modification — is recorded as an execution event. This event feeds back into the knowledge validation pipeline:

- **Acceptance** — corroborates the temporal pattern, increasing confidence
- **Denial** — counts as contradictory evidence; repeated denials reduce confidence and eventually invalidate the pattern
- **Modification** — updates the temporal expression (e.g., "not now, do it at 8 AM instead") and adjusts the schedule

This creates a self-correcting automation system. Patterns that serve the user persist and strengthen; patterns that don't are pruned through natural interaction. No explicit configuration is required — the system learns when to act and when to stop from lived experience.

**Invariant clarification:** The core invariant "AI proposes intent; the kernel executes deterministically" (Section 1.4) is preserved by the temporal intent generator. Temporal intent is generated by a _deterministic kernel component_ (the temporal intent generator) from _validated knowledge_ — not by the RPU. The knowledge that triggers temporal intent has passed through the validation pipeline (corroboration, contradiction detection, confidence scoring). The temporal intent generator is a scheduler, not a reasoning engine. It monitors the knowledge graph for active patterns and produces intent when conditions are met, using deterministic schedule matching. No AI reasoning is involved in the generation step.

### 8.3 Indexing Strategy

Each sub-graph uses different indexing strategies appropriate to its domain:

The perception graph indexes by spatial location, time window, sensor type, and derived semantic tags. This enables queries like "what did the system perceive in this room during this time period."

The execution graph indexes by chain ID, priority tier, tool type, and outcome status. This enables queries like "what chains failed in the last hour" or "which tools are most frequently used."

The memory graph indexes by semantic similarity, temporal proximity, emotional valence, and relationship context. This enables queries like "what past experiences are relevant to the current situation."

The knowledge graph indexes by semantic topic, confidence score, validation status, source provenance, and contradiction history. This enables queries like "what facts are known about topic X and how confident are we."

The summarization pipeline is what makes the six-graph model operationally viable. Without it, queries across the full event log would be prohibitively expensive. With it, each graph operates on appropriately summarized data, and the memory and knowledge graphs emerge naturally from the pipeline at different compression levels.

The pipeline runs continuously, with summarization depth adjustable based on compute availability. In dynamic repartition mode, the offline optimization loop can re-summarize historical situation model with improved algorithms, updating the memory and knowledge graphs without touching the immutable event log.

---

## 9. Derived Graphs

The macro graph is a derived graph — a compiled, validated structure built from execution traces. The code registry shares the same event-sourced model but serves a different purpose: it is a repository of tested code components that agents use when writing software, not a runtime optimization.

**Macros are for the runtime. The code registry is for software development. They never intersect.**

Both systems share TypeScript + Effect as a common primitive vocabulary and the same Git-like, event-sourced storage model. This is an implementation convenience, not a conceptual unity.

### 9.1 Macro System

Macros are compiled execution subgraphs that capture repeated behavior as deterministic conditional logic and tool calls, annotated with Reasoning Hints. A macro is not a reasoning engine — it executes to completion as a deterministic piece of the execution graph.

Macros use Tool Services only — they never use Application Services.

Macros are runtime-only — they execute in the TypeScript/Effect runtime and are not transpiled.

#### 9.1.1 Reasoning Hints

A Reasoning Hint is a distilled textual annotation written by the model during macro compilation. It captures the outcome of a reasoning step at a branch point, not the full chain-of-thought that produced it. Hints serve three purposes: provenance (explaining why a branch was taken), context injection (passed to the RPU as a sliding window of recent decisions), and future learning (giving macro discovery situation model context to work with).

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

Repeated execution patterns are detected through sliding window mining over event streams, subgraph matching in trace graphs, and normalization of tool calls. The macro system operates as a reversible execution compression pipeline.

The process follows three steps. A macro is proposed, assisted by LLM analysis of trace patterns. It is validated through tests against historical execution data. Once validated, it is compiled into a reusable execution unit. It can be demoted if it falls out of use or if underlying primitives change.

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

These criteria prevent the system from generating low-frequency, brittle, or marginally useful patterns that add complexity without value.

#### 9.1.4 Discovery as an Open Problem

Detecting useful reusable patterns is fundamentally difficult. The macro system's discovery mechanism is an active area of development. The criteria above provide a starting framework, but the actual mining algorithms, subgraph matching strategies, and situation model equivalence detection remain open research questions. The system is designed to evolve its discovery mechanism without changing the macro compilation, validation, or execution model.

### 9.2 Code Registry

The code registry is **not part of the runtime's execution path**. It is a repository of tested, versioned code components that the model, agents, and users draw from when writing software together. When the system helps a user build an application, it pulls from the registry instead of generating code from scratch.

The code registry and the runtime's orchestration layer share TypeScript + Effect as a common primitive vocabulary for transpilation, but they never intersect. Macros compose Tool Services at runtime. Code components compose Application Services during software development.

**The problem.** Most application code consists of common patterns: data transformation, validation, formatting, state management, error handling. These are repeatedly written, rarely tested thoroughly, and pulled from external packages that may carry security risks or unnecessary dependencies.

**The solution.** Build an npm-like repository where code components are written once, tested thoroughly, and versioned. The model pulls from the registry when writing new code instead of reinventing logic. Every component has a full history — who wrote it, what tests it passes, what depends on it, how it evolved. Quality improves through reuse: frequently used components get more testing, edge cases get discovered, they get refined. Safety comes from versioning: you know exactly what code is included in generated programs, it's been validated, and you can roll back.

Code components are Effect-style function definitions that compose Application Services through semantic orchestration primitives. Code components use Application Services only — they never use Tool Services.

Code components support transpilation across language runtimes. TypeScript + Effect transpiles nearly 1:1 to Go due to structural similarity in their concurrency models — goroutines, contexts, and channels map closely to Effect primitives. C++ requires a runtime layer to support the same semantic primitives, reserved for environments like Unreal Engine that expect native code. Native code modules can be exposed as Application Services, enabling cross-language composition.

The code registry applies the same Git-like, event-sourced model used for world-state and macros to the domain of versioned code components. It is a homelab-hosted repository where every piece of code — written by the model, by autonomous agents, or by the user — is tested, versioned, and stored as a reusable, content-addressed artifact.

#### 9.2.1 Relationship to Macros

Macros and code components are two applications of the same underlying pattern: repeated behavior is captured, validated, and made reusable. The distinction is in discovery and validation.

| Runtime               | Software                   |
| --------------------- | -------------------------- |
| Macro Graph           | Code Registry              |
| Execution compression | Implementation compression |
| Passive discovery     | Active authoring           |
| Historical validation | Test validation            |

Macros are passively discovered through trace mining and validated against historical execution data. They compress the runtime's own execution patterns.

Code components are actively written and must pass a mandatory test pipeline before adoption. They compress implementation patterns used in software the system helps users build.

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

**10. Temporal Intent** — The temporal intent generator monitors the knowledge graph for active temporal patterns. When the schedule fires at 7 AM, it produces an intent: `{ type: "intent", source: "temporal", desire: "get weather forecast", triggered_by: "knowledge_xyz", schedule_type: "recurring" }`. This flows through the same proposal → commit pipeline. The user's response — acceptance or denial — feeds back into knowledge, adjusting confidence or invalidating the pattern.

**11. Compression** — If a macro exists for the weather check pattern, the temporal intent triggers the macro instead of full reasoning. The macro executes deterministically with a Reasoning Hint: "User typically wants weather at 7 AM, proactively offering."

Each step is an immutable event in the world-state graph. The full provenance chain is traceable. The system can answer: why was the weather fetched, what perception triggered it, what situation model was generated, what chain executed it, and what knowledge was derived.

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
