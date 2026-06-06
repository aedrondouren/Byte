# Distributed Cognitive Runtime

**Technical Concept Document**

---

## Related Documents

- [GRAPH.md](core/GRAPH.md) — Git-like world-state graph model (immutable events, DAG, provenance, 6-graph model)
- [RPU.md](core/RPU.md) — Reasoning Processing Unit coprocessor pattern
- [EDGE.md](core/EDGE.md) — Portable Personal Edge Node architecture
- [INTERFACE.md](core/INTERFACE.md) — Multimodal cognitive interface
- [ORCHESTRATION.md](core/ORCHESTRATION.md) — Semantic orchestration primitives and service ecosystems
- [MACROS.md](core/MACROS.md) — Macro system (Tool Services, runtime-only)
- [REGISTRY.md](core/REGISTRY.md) — Code registry (Application Services with transpilation)
- [CORE.md](core/CORE.md) — Condensed reference summary

---

## 1. Overview

A distributed, event-sourced cognitive runtime where all perception, reasoning, tool execution, and media output are unified into a single canonical world-state graph, continuously optimized via deterministic execution tracing and reversible compilation of behavior into reusable semantic macros.

This system replaces agent loops, prompt chains, and tool-using chatbots with a persistent execution runtime over a shared world-state graph. Everything — LLM inference, sensor inputs, tool calls, streaming, automation — becomes a projection or transformation of that graph.

The world-state graph is a Git-like, content-addressed history of cognition. Every perception, reasoning step, tool call, scheduler decision, and state transition is recorded as an immutable event in a causally linked DAG. Current state is not stored separately; it is a projection of the event history.

The core philosophical invariant: the system is not a chatbot, not an agent, and not a framework. It is a persistent, distributed, event-sourced execution substrate where cognition, perception, and action are unified through a canonical world-state graph.

The system's unifying principle: the runtime continuously converts high-entropy events into low-entropy reusable structure, reducing the amount of reasoning required to operate over time. This principle manifests in every layer — the RPU minimizes reasoning through accumulated structure, the summarization pipeline compresses semantic into narrative memories, the macro system compiles repeated execution patterns, the code registry captures tested reusable components, and the knowledge graph crystallizes validated facts from experience.

---

## 2. Core Invariants

The following principles are non-negotiable and govern every design decision in the system.

**AI proposes intent; the kernel executes deterministically.** The LLM never runs the system. It proposes structured intent into the execution kernel, which validates, schedules, and executes it using deterministic primitives.

**Everything is an event in the world-state graph.** There is no distinction between input and output. Sensor readings, tool calls, reasoning steps, execution chains, and environmental state are all events in the same canonical structure.

**The world-state is an immutable event graph, not a mutable database.** Events are append-only, content-addressed, and cryptographically verifiable. Current state is always a projection of history, never the source of truth.

**Execution chains are persistent, prioritized, and interruptible.** Chains survive waiting, suspension, and resumption. They are governed by priority levels and resource quotas, and non-critical chains can be interrupted while critical safety chains cannot.

**Macros are compiled execution subgraphs, not behavior overrides.** Repeated execution patterns are discovered, validated, and compiled into reusable units. They can be demoted if unused. They never replace primitives.

**Edge-cloud separation of cognition.** Real-time perception and immediate interaction logic run on edge devices. Heavy reasoning, long-term memory, and planning run on the homelab. The two layers communicate through structured events, not raw media.

**AI is an interpretation layer, not a control system.** Infrastructure and deterministic logic never depend on AI reasoning. AI interprets fused sensor data and proposes actions; the kernel decides whether and how to execute them.

**Reasoning is externalized into accumulated structure.** The objective is not to maximize model intelligence but to minimize the amount of reasoning required to produce intelligent behavior. Repeated reasoning is transformed into persistent state, projections, plans, memories, and reusable cognitive artifacts.

**Graceful degradation is mandatory.** The system must remain functional and safe when disconnected from the homelab, when network links fail, or when sensor inputs become unreliable.

---

## 3. World-State Graph Architecture

The world-state graph is the single source of truth for the entire system. It is a canonical event graph where every piece of system activity is recorded as a structured event with normalized arguments, timestamps, and dependency edges.

IR stands for Intermediate Representation — a canonical, normalized event format that all system components read and write. The Trace IR is the specific intermediate representation used for execution. It decouples event producers (LLMs, sensors, tools) from event consumers (scheduler, macro system, observability pipeline), enabling each component to operate independently without knowledge of the others' internal formats.

Nothing is treated as input versus output. Everything is an event in the same system.

### 3.1 Sub-Graph Architecture

Conceptually the world-state is unified. Operationally it is split into six logical graphs to keep queries simple, indexing efficient, and maintenance manageable. Each graph follows the same Git-like model — immutable events, content-addressed, DAG-structured, state-as-projection — but handles a different domain.

**Perception Graph** — structured perception from signal processing systems, environmental facts, user state, biometrics. Answers "what is happening in the world." Contains the outputs of object detection, speech-to-text, attention tracking, movement analysis, and cognitive state processing. These are the first entries in the graph — raw sensor streams never enter.

**Execution Graph** — chains, tasks, scheduling decisions, tool calls, RPU invocations. Answers "what is the system doing." Contains execution chain lifecycle, scheduler decisions, tool call records, RPU request/response pairs, and state transitions.

**Memory Graph** — episodic experiences, emotional context, narrative memories, personality evolution, relationship state. Answers "what has been experienced." Retrieved by similarity, temporal proximity, and emotional valence. Evolves as new experiences accumulate and context shifts.

**Knowledge Graph** — validated facts, semantic knowledge, tested relationships, crystallized patterns. Answers "what is known to be true." Retrieved by semantic query. Facts must pass a validation pipeline before becoming knowledge. Knowledge carries temporal validity, confidence decay, and revision chains — it is durable but not final.

**Macro Graph** — compiled execution subgraphs. Discovery is passive through sliding window mining. Validation tests against historical traces. Answers "what patterns repeat and whether they are valid."

**Code Registry Graph** — versioned code components, tests, and dependencies. Components are actively written by model, agents, or user. A mandatory test pipeline validates before adoption. Answers "what code exists, what version is current, and whether it passes tests."

Each graph maintains its own projection layer for queries, its own indexing strategy, and its own retention policies. The event log remains the source of truth for all six. Channels attach to these projections to expose state to external surfaces.

### 3.2 Cross-References Between Sub-Graphs

Graphs reference each other through content-addressed hashes. A perception (dog detected) can be referenced by execution (chain triggered object interaction). An execution outcome (task completed) can be referenced by memory (experience indexed). A memory lookup (recall past interaction) can be referenced by execution (context injected into RPU request). A validated fact (Bob prefers Rust) can be referenced by execution (code generation chose Rust for Bob's project). Cross-graph references work like git submodules — content-addressed hashes link code components to macro bodies, macro executions to world-state, test runs to the execution graph, memory lookups to perception and execution context, and knowledge facts to any graph that requires validated information.

### 3.3 Events as Immutable Commits

Every event is immutable. Events reference their causal parents, are cryptographically hashed and optionally signed, and the history is append-only. This creates a tamper-evident record of all system activity.

### 3.4 State as Projection

Current state is derived from replaying events. State is a materialized view, not the source of truth. Checkpoints accelerate reconstruction but do not replace history. The system can always rebuild current world-state from the full event log.

### 3.5 Projections and Channels

The runtime distinguishes between two categories of transformation over the event graph:

**Projections** transform graph state into graph state. They are internal runtime transformations that produce durable, queryable, composable knowledge structures. A projection is a pure function over an event stream:

```
Graph → Projection → Graph
```

Examples:

```
Perception → Perception Graph
Perception Graph → Semantic
Semantic + User Signals → Intent
Intent → Execution Graph
Execution Graph → Memory Graph
Memory Graph → Knowledge Graph
Execution Graph → Macro Graph
```

Projections optimize for correctness, queryability, persistence, and composition. They can be chained — the output of one projection becomes the input of another. The summarization pipeline (Section 9) is a chain of projections: semantic into narrative memories, into validated knowledge.

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

### 3.5.1 Projection Types

Projections are classified by their determinism level. This classification governs how they are debugged, replayed, and versioned.

**Deterministic projections** produce identical output given identical input. They are fully replayable, versioned by their logic hash, and form the backbone of kernel-level state derivation. Examples: perception processing (object detection, speech-to-text), execution graph derivation, chain lifecycle state projection.

**Probabilistic projections** produce output that may vary across replays due to LLM involvement. They carry confidence scores, are versioned by both logic hash and model version, and require evidence trails for debugging. Examples: semantic interpretation, intent estimation from fused signals.

**Hybrid projections** combine deterministic and probabilistic stages. The deterministic stages are fully replayable; the probabilistic stages carry confidence and model version metadata. Examples: summarization pipeline (deterministic aggregation + LLM-assisted narrative generation), knowledge validation (deterministic corroboration + LLM-assisted contradiction detection).

This classification matters for debugging. When state is incorrect, the projection type determines the investigation path: deterministic projections are debugged by replaying input; probabilistic projections are debugged by examining evidence trails and confidence scores; hybrid projections are debugged by isolating which stage produced the error.

### 3.6 DAG Structure

The graph is a DAG, not a linear chain. Multiple chains execute concurrently. Reasoning can fork. Execution paths can merge. Lineage is preserved across all branches. Execution chains behave like branches — long-running tasks become branches of execution that can pause, resume, fork, merge, or terminate. The scheduler determines which branches receive resources.

### 3.7 Provenance

Provenance is first-class. Every action is traceable to its origin: perception processing leads to world-state version, which leads to semantic interpretation, which leads to intent proposal, which leads to scheduler decision, which leads to tool execution. The system can answer what happened, why it happened, what information was available, what alternatives existed, and which chain caused it.

### 3.8 Git, Not Blockchain

The goal is not distributed consensus. The goal is tamper-evident history, causal lineage, replayability, auditability, and reproducibility. The architecture is closer to Git plus Event Sourcing plus Knowledge Graph plus Distributed Runtime than to a blockchain.

---

## 4. Signal-to-Intent Pipeline

Raw sensor streams never enter the world-state graph. Camera feeds, audio streams, gaze tracking, IMU readings, and EEG signals are piped directly to specialized processing systems. The first entries in the graph are structured perception — the outputs of those systems.

The pipeline converts raw signals into actionable intent through three layers. Each layer produces structured data that is recorded in the event log for provenance, model training, and system improvement.

### 4.1 Perception Processing (Signal → Structure)

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

### 4.2 Semantic Interpretation (Structure → Meaning)

Perception is cross-modal interpretation. It combines multiple perception types into coherent understanding of what is happening.

Semantic interpretation runs on the homelab and uses the RPU pattern: structured JSON input produces structured JSON output. The model receives batched windows of perception (e.g., 30-second windows) along with context and personality state, and produces semantic interpretation with confidence scores and evidence trails.

The model does not generate free-form text. It receives a deterministic prompt with a fixed output schema and returns structured semantic. This is the same RPU contract used for reasoning, applied to perception interpretation.

Semantic interpretation is LLM-assisted and the LLM is a cornerstone of the system. Without it, semantic interpretation falls back to rule-based heuristics with significantly reduced fidelity. Perception processing continues regardless — the system degrades but does not fail.

Example semantic:

```json
{
  "type": "semantic",
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

Semantic is recorded in the event log but is an intermediate transform — it feeds into intent estimation rather than becoming a destination graph.

### 4.3 Intent Estimation (Meaning → Action)

Intent estimation converges two sources into a unified intent format:

**User intent** — explicit signals from the multimodal layer. Voice provides explicit commands. Gaze provides attention and selection. EMG and blink provide confirmation. EEG provides state modulation. These signals are fused into user intent.

**System intent** — autonomous proposals generated from semantic and context. When semantic indicates the user has been stuck on a problem for 30 minutes, the system may propose offering help. When semantic indicates the user is cooking, the system may propose pulling up a recipe. These are system-generated intent proposals.

Both sources converge into the same intent format. Intent flows through three stages before execution:

**Intent** — a state change request or desire. Originates from user signals or system-generated semantic interpretation. Carries confidence, context, and evidence trails. Does not specify how to execute.

**Proposal** — an executable candidate derived from intent. The kernel or RPU translates intent into a concrete action plan with specific tool calls, chain triggers, or state updates. Proposals are validated against current world-state before proceeding.

**Commit** — a kernel-approved execution decision. The scheduler validates the proposal against resource availability, priority constraints, and safety rules. Once committed, the proposal becomes an execution chain in the execution graph.

This three-stage flow separates concerns: intent expresses what is desired, proposal specifies how to do it, commit authorizes execution. The kernel does not distinguish between user-originated and system-originated intent at the intent stage — both flow through the same validation, proposal generation, and commit pipeline.

Example intent:

```json
{
  "type": "intent",
  "source": "system",
  "desire": "offer assistance with current task",
  "confidence": 0.78,
  "triggered_by": "sem_xyz789",
  "context": {
    "semantic": "user is in focused work session",
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

### 4.4 Reprocessability

Because perception is preserved in the event log, the entire downstream pipeline can be reprocessed. Semantic interpretation can be re-run with better models. Intent estimation can be re-evaluated with improved context understanding. This enables continuous system improvement without losing historical data.

The offline optimization loop (Section 12) reprocesses historical perception to generate improved semantic and intent, which updates the memory and knowledge graphs without modifying the immutable perception log.

---

## 5. Cognitive Runtime Kernel

The kernel is the irreplaceable core of the architecture. It is the permanent substrate — the component that cannot be swapped out or replaced without rebuilding the entire system. While the RPU can be exchanged for any reasoning engine, the kernel governs all execution.

It manages execution chains as DAG-based computation graphs rather than linear sequences. It handles scheduling, prioritization, interruption and resumption, long-lived workflows, resource arbitration, and tool execution lifecycle.

The kernel receives structured intent proposals from the LLM and other sources, validates them against current world-state, schedules them according to priority and resource availability, and executes them using the semantic orchestration primitives defined in the compatibility layer.

### 5.1 Non-Graph Runtime Layer

The scheduler, execution workers, cache system, and resource arbitrator are not part of the world-state graph. They operate over the graph, not inside it. These components are ephemeral runtime infrastructure — they manage execution flow, allocate compute, and maintain in-memory state for active chains. They do not produce graph entries directly; they produce execution outcomes that become events in the execution graph.

This distinction matters for debugging and system boundaries. The world-state graph is the persisted, replayable record of everything that happened. The non-graph runtime layer is the machinery that makes things happen — it is transient, replaceable, and not subject to replay. If the kernel restarts, the non-graph layer rebuilds from the current graph state. If a projection is re-run, the non-graph layer executes it but does not become part of its output.

### 5.2 Execution Model

The system runs on a unified abstraction: an execution chain is a persistent, prioritized, interruptible cognitive process.

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

The kernel invokes the Reasoning Processing Unit as a specialized coprocessor, not as an autonomous agent. The RPU receives structured requests containing world-state projections, personality state, context, and task definitions. It returns structured cognitive artifacts — plans, summaries, state update proposals, and action proposals — rather than raw conversational text.

### 5.4 Planning-First Execution

Long-running tasks follow a planning-first workflow. The user request triggers an acknowledgement, then plan generation by the RPU, then execution, then a recap, then the conversation response. The plan becomes an observable runtime artifact. Execution updates the plan state rather than generating arbitrary conversational messages. This creates deterministic visibility into system progress.

### 5.5 Recap-Based Observability

The runtime maintains execution state independently from the conversation. Users may request current status, task progress, plan state, or execution recap. The response is generated from runtime projections rather than reconstructed from conversation history. This avoids using conversational messages as the source of truth.

---

## 6. Reasoning Processing Unit

The RPU treats a Large Language Model as a specialized reasoning coprocessor rather than as a complete autonomous agent. Instead of embedding memory, planning, identity, world state, execution management, and communication into a single prompt, these concerns are externalized into dedicated runtime systems. The model becomes a deterministic cognitive transformation engine operating over structured state.

The objective is not to maximize model intelligence, but to minimize the amount of reasoning required to produce intelligent behavior. Intelligence is the ability to accumulate structure so that less reasoning is required in the future.

### 6.1 RPU Responsibilities

The RPU is responsible for: interpretation, reasoning, synthesis, planning, summarization, reflection, communication generation, and cognitive transformations.

The RPU is not responsible for: memory persistence, event storage, scheduling, state tracking, personality storage, tool orchestration, world modeling, or execution management. These functions belong to the runtime.

### 6.2 Cognitive Contract

Every inference executes through a structured contract. The RPU receives a request containing the function to perform, the objective, personality state, world state, context projection, and optionally task state and previous artifacts. It returns a result along with optional summary, plan, memory suggestions, state updates, next actions, and metadata including confidence and reasoning mode.

As an example of the form this takes, a TypeScript interface might look like:

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

The RPU produces structured cognitive artifacts rather than raw conversational text.

### 6.3 Personality as State

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

### 6.4 Cognitive Cache Hierarchy

The architecture operates as a hierarchy of increasingly expensive cognitive operations:

- L1 — Active Context
- L2 — Structured State
- L3 — World Model
- L4 — Event History
- L5 — Reasoning (RPU)

Reasoning is the most expensive resource. The purpose of the runtime is to convert reasoning into reusable structure and prevent unnecessary recomputation.

### 6.5 Guiding Question

The central design question of the RPU architecture is: "What can be removed from the reasoning loop?" Any responsibility that can be represented as deterministic state, stored structure, or runtime infrastructure should be externalized from the model. The RPU should only perform transformations that genuinely require reasoning.

### 6.6 Model Independence

The RPU abstraction enables model interchangeability. The runtime defines state, contracts, events, personality, memory, and scheduling. The model only implements reasoning functions. This allows multiple model classes to participate in the same cognitive runtime.

### 6.7 LLM Runtime

A standalone adapter layer that standardizes heterogeneous LLM providers into a universal runtime representation. It adapts any model — OpenAI-compatible, native APIs, or custom backends — to the same normalized IR schema.

Responsibilities include normalizing streaming inference outputs, translating tool-call formats across providers, preserving reasoning and structured outputs when available, and decoupling providers from downstream systems.

The LLM Runtime ensures that the cognitive runtime kernel never needs to know which specific provider is backing a given inference request. All providers emit the same IR format. This layer is about model adaptation only — it does not use semantic code components or orchestration primitives.

### 6.8 Event-Sourced Cognition

All runtime actions generate events. The event log becomes the canonical source of truth. The conversation is merely one channel over runtime state — a projection exposed through a communication surface.

---

## 7. Semantic Orchestration Layer

The system defines a minimal set of Effect-like orchestration primitives that are semantically equivalent across all runtime backends. These primitives define what a program means, independent of any programming language.

Core primitives include run, fork, parallel, race, scope, acquire, release, fail, recover, mapError, stream, pipe, sink, provide, layer, and context. These are not a domain-specific language. They are a portable execution meaning layer.

Above the primitives sits a service-based composition layer. Services define the leaves of execution, while the core workflow logic remains pure and portable. The system maintains two separate service ecosystems that never intersect:

### 7.1 Tool Services

Tool Services are exposed to macros. Individual tool calls are exposed as service methods. Macros compose these services through semantic orchestration primitives to execute automations — combining tool calls with conditional logic to capture reasoning and tool usage patterns.

Tool Services are runtime-only. They are not transpilable and exist only within the TypeScript/Effect runtime.

### 7.2 Application Services

Application Services are exposed to code components. These are application-level capabilities — databases, APIs, storage, and native code modules. Code components compose these services through semantic orchestration primitives.

Application Services support transpilation across language runtimes. TypeScript + Effect transpiles nearly 1:1 to Go due to structural similarity in their concurrency models. C++ requires a runtime layer to support the same semantic primitives, reserved for environments like Unreal Engine that expect native code. Native code modules can be exposed as Application Services, enabling cross-language composition.

Code components use Application Services only — they never use Tool Services.

### 7.3 Consumers

These primitives are consumed by two derived graph types (Section 8):

- **Macros** use Tool Services only, runtime-only, no transpilation
- **Code components** use Application Services only, with transpilation support

The two ecosystems never intersect.

---

## 8. Derived Graphs

The macro graph and code registry graph are derived graphs — they are not primary sources of activity but rather compiled, validated structures built from the primary graphs. Both follow the same underlying pattern: repeated behavior is captured, validated, and made reusable. They live in separate logical graphs to keep queries simple and maintenance manageable.

### 8.1 Macro System

Macros are Effect-style function definitions that compose Tool Services through semantic orchestration primitives. Individual tool calls are exposed as service methods; macros combine them with conditional logic to capture reasoning and tool usage patterns. Macros use Tool Services only — they never use Application Services.

Macros are runtime-only — they execute in the TypeScript/Effect runtime and are not transpiled.

Repeated execution patterns are detected through sliding window mining over event streams, subgraph matching in trace graphs, and normalization of tool calls. The macro system operates as a reversible execution compression pipeline.

The process follows three steps. A macro is proposed, assisted by LLM analysis of trace patterns. It is validated through tests against historical execution data. Once validated, it is compiled into a reusable execution unit. It can be demoted if it falls out of use or if underlying primitives change.

Macros are compiled execution subgraphs, not behavior overrides. They are an optimization layer for execution efficiency and reliability. They never replace primitives, and they remain fully reversible.

#### 8.1.1 Macros as Compiled Commit Ranges

Frequently occurring subgraphs are identified, validated, and compiled. A macro can always be expanded back into its originating events. This guarantees that no behavior is ever hidden behind macro abstraction — the full provenance chain remains accessible.

#### 8.1.2 Macro Promotion Criteria

Not every repeated pattern should become a macro. A pattern is promoted only when it meets multiple criteria:

- **Frequency** — the pattern occurs often enough that compilation yields measurable efficiency gains
- **Success rate** — the pattern completes successfully across diverse contexts, not just in narrow conditions
- **Stability** — the underlying primitives and data shapes are unlikely to change in ways that would invalidate the macro
- **Cost savings** — the macro reduces compute time, token usage, or resource consumption compared to raw execution
- **Semantic equivalence** — the macro's behavior is functionally identical to the original pattern, not an approximation

These criteria prevent the system from generating mountains of garbage macros — low-frequency, brittle, or marginally useful patterns that add complexity without value.

#### 8.1.3 Discovery as an Open Problem

Detecting useful reusable patterns is fundamentally difficult. The macro system's discovery mechanism is an active area of development. The criteria above provide a starting framework, but the actual mining algorithms, subgraph matching strategies, and semantic equivalence detection remain open research questions. The system is designed to evolve its discovery mechanism without changing the macro compilation, validation, or execution model.

### 8.2 Code Registry

Code components are Effect-style function definitions that compose Application Services through semantic orchestration primitives. Code components use Application Services only — they never use Tool Services.

Code components support transpilation across language runtimes. TypeScript + Effect transpiles nearly 1:1 to Go due to structural similarity in their concurrency models. C++ requires a runtime layer to support the same semantic primitives, reserved for environments like Unreal Engine that expect native code. Native code modules can be exposed as Application Services, enabling cross-language composition.

The code registry applies the same Git-like, event-sourced model used for world-state and macros to the domain of versioned code components. It is a homelab-hosted repository where every piece of code — written by the model, by autonomous agents, or by the user — is tested, versioned, and stored as a reusable, content-addressed artifact.

Over time, this builds an in-house library that replaces external dependencies for small logic. Core packages (frameworks, bundlers, compilers) remain as dependencies, but business logic, common patterns, and utility code are internal, tested, and versioned within the registry.

#### 8.2.1 Relationship to Macros

Macros and code components are two applications of the same underlying pattern: repeated behavior is captured, validated, and made reusable. The distinction is in discovery and validation.

Macros are passively discovered through trace mining and validated against historical execution data. They compress execution patterns.

Code components are actively written and must pass a mandatory test pipeline before adoption. They compress implementation patterns.

Both follow the "history is more important than current state" principle. Both are versioned, reversible, and traceable.

#### 8.2.2 Mandatory Test Pipeline

Unlike macros, which are passively discovered and validated against historical data, code components must actively pass a test pipeline before they are adopted into the registry. Every new version of a code component must:

- Pass its full test suite
- Maintain or improve coverage thresholds
- Not introduce regressions in dependent components
- Be compatible with declared dependencies

The test pipeline runs in the homelab's trusted environment. Only components that pass all checks are published to the registry. Failed versions are recorded in the event log but never become available for use.

#### 8.2.3 Homelab as Trusted Source

The code registry lives on the homelab as the trusted source of truth. Edge nodes pull from it when needed — initially rarely, but as the system grows, edge devices including small laptop setups and the PEN may pull tested components for local execution.

The registry is versioned and content-addressed. Edge nodes pin to specific versions and can verify integrity through cryptographic hashes. No untested or unversioned code is ever distributed.

#### 8.2.4 Goal: In-House Logic, Minimal External Dependencies

The long-term objective is to replace external dependencies for small logic with tested, versioned, in-house components. Most application code consists of common patterns — data transformation, validation, formatting, state management, error handling — that can be captured, tested, and reused.

Core infrastructure packages (frameworks, bundlers, compilers, database drivers) will always be external. But the logic that glues them together becomes internal, auditable, and continuously improving through reuse.

---

## 9. Event Summarization and Indexing

The signal-to-intent pipeline (Section 4) produces perception, semantic, and intent at high frequency. The summarization pipeline operates downstream of intent, converting accumulated experience into progressively more compact and meaningful representations suitable for long-term storage and retrieval.

Without summarization, the system would drown in accumulated experience. The summarization pipeline converts high-volume semantic into progressively more compact narrative memories and validated knowledge.

### 9.1 Summarization Pipeline

The event log is the complete, unfiltered record of all perception, semantic, intent, execution, and system state transitions. This layer is append-only and immutable. It is the source of truth but not a query surface.

Narrative memories are consolidated experiences indexed for long-term recall. A sequence of semantic from a trip becomes "visited location Y, encountered situation Z, took action W, outcome was positive." Narrative memories populate the memory graph and serve as context for future RPU invocations.

Validated knowledge is extracted from semantic and narrative memories through corroboration, testing against conflicting evidence, and consistency checks. A fact becomes knowledge only when it survives repeated validation across independent events. Facts held in a "hypothesis" state when contradictory evidence exists. Once a fact reaches sufficient confidence, it crystallizes into the knowledge graph.

### 9.2 Knowledge Time Semantics

Knowledge is not final. It carries temporal semantics:

**Temporal validity** — knowledge is valid within a time window. "User prefers Rust" may be true for a project phase but not indefinitely. Knowledge entries carry `valid_from` and `valid_until` timestamps. When `valid_until` passes without corroboration, confidence decays.

**Confidence decay** — knowledge that is not corroborated over time loses confidence. The decay rate depends on the knowledge domain: technical facts decay slowly, social facts decay faster, environmental facts decay fastest. When confidence drops below the crystallization threshold, knowledge reverts to hypothesis status.

**Revision chains** — when knowledge is contradicted, the old version is not deleted. It is superseded by a new version with a causal link to the contradiction. The full revision history is preserved in the knowledge graph, following the same Git-like versioning model as all other graphs. A fact can be true at time T and false at time T+1 — both versions coexist with temporal validity windows.

### 9.3 Indexing Strategy

Each sub-graph uses different indexing strategies appropriate to its domain:

The perception graph indexes by spatial location, time window, sensor type, and derived semantic tags. This enables queries like "what did the system perceive in this room during this time period."

The execution graph indexes by chain ID, priority tier, tool type, and outcome status. This enables queries like "what chains failed in the last hour" or "which tools are most frequently used."

The memory graph indexes by semantic similarity, temporal proximity, emotional valence, and relationship context. This enables queries like "what past experiences are relevant to the current situation."

The knowledge graph indexes by semantic topic, confidence score, validation status, source provenance, and contradiction history. This enables queries like "what facts are known about topic X and how confident are we."

### 9.4 Role in the Architecture

The summarization pipeline is what makes the six-graph model operationally viable. Without it, queries across the full event log would be prohibitively expensive. With it, each graph operates on appropriately summarized data, and the memory and knowledge graphs emerge naturally from the pipeline at different compression levels.

The pipeline is itself a chain of projections. Each stage — semantic to narrative memories, narrative memories to validated knowledge — is a projection transforming graph state into graph state. Upstream processing (perception generation, semantic interpretation, intent estimation) is handled by the signal-to-intent pipeline (Section 4).

The pipeline runs continuously, with summarization depth adjustable based on compute availability. In dynamic repartition mode, the offline optimization loop can re-summarize historical semantic with improved algorithms, updating the memory and knowledge graphs without touching the immutable event log.

---

## 10. Edge Architecture

The edge architecture extends the world-state graph spatially through mobile and fixed sensor nodes. It consists of two physical components.

The home system provides fixed cameras, environmental mapping, persistent spatial memory, and the homelab cognitive core. It is the anchor point for all long-term reasoning and memory.

The wearable backpack node — the Portable Personal Edge Node, or PEN — provides mobile sensors, multi-uplink networking, live environmental injection, and extends the world-state beyond the home boundary. It is a battery-powered distributed computing system consisting of a wearable compute core connected to heterogeneous sensor devices including phones, glasses, cameras, optional depth sensors, and biometric inputs.

The system does not observe the world. It continuously extends its world-state graph through mobile perception nodes.

### 10.1 Edge Perception Layer

Real-time multi-camera egocentric capture, depth sensing and IMU tracking, audio capture and translation, local object detection and scene parsing. Produces structured perception — the first entries in the world-state graph. Raw sensor streams never leave the edge; only structured perception enters the graph.

### 10.2 Local Deterministic Compute Layer

Low-latency inference models, sensor fusion and SLAM, networking and routing, policy enforcement, AR rendering and interaction logic. Ensures the system remains stable even when offline.

### 10.3 Remote Cognitive Layer (Homelab)

High-capacity reasoning and planning, long-term memory and trip reconstruction, scene re-synthesis through 3D splats or neural reconstruction, streaming production and content orchestration, identity, personality, and dialogue continuity.

### 10.4 Network Orchestration Layer

Multi-WAN aggregation across LTE, Wi-Fi, and Ethernet. VPN tunnel to home as the primary trusted egress. Policy-based routing balancing latency versus trust versus cost. Always-on heartbeat maintaining global state continuity.

### 10.5 Memory and World Model

Continuous heartbeat telemetry stream. Event-based logging rather than raw media storage. Spatial reconstruction of visited environments. Semantic indexing of experiences over time. Optional 3D re-renderable experience bubbles.

The system functions as a personal mobile perception and cognition layer, enabling augmented navigation and awareness, live contextual translation and information overlay, distributed multi-camera IRL streaming, reconstructable spatial memory of lived environments, and continuous synchronization with a home-based intelligence core.

---

## 11. Multimodal Cognitive Interface

The system is a world-state engine that continuously updates a structured representation of the environment, user attention, and internal state. Inputs come from multiple modalities — voice, gaze tracking, IMU and head pose, cameras in RGB, depth, and IR, EEG providing low-bandwidth cognitive and context signals, and EMG or micro-gestures for confirmation.

Each sensor contributes partial, noisy evidence that is fused into a unified real-time intent and context graph.

### 11.1 Modalities

Gaze provides attention and selection signals. Voice provides explicit intent. EMG and blink signals provide confirmation. EEG provides state modulation including fatigue, stress, and cognitive load. Behavior history provides priors for intent estimation. Personality state modulates how the system interprets and responds to fused intent.

Interaction is probabilistic intent estimation, not deterministic control. No single modality is sufficient on its own. The system fuses partial, noisy evidence from multiple modalities to estimate what the user intends, and the kernel validates and executes only when confidence thresholds are met.

### 11.2 Intent Fusion

Human inputs are not commands. They are probabilistic signals fused into a unified intent and context graph. User intent from the multimodal layer converges with system-generated intent from semantic interpretation, both flowing through the same intent → proposal → commit pipeline (Section 4.3).

### 11.3 AR Interface as Output Channel

The AR interface acts as the primary output channel, turning the internal world model into spatially grounded overlays, notifications, and adaptive UI elements.

EEG and other biosignals do not act as direct command channels. They are contextual modulation signals that adjust system behavior based on attention, fatigue, stress, or engagement, improving the timing and relevance of interactions.

The hierarchical control stack separates concerns by latency. Low-latency wearable or backpack compute handles perception, signal filtering, and immediate interaction logic. The homelab system provides heavier reasoning, long-term memory, planning, and tool execution. A custom orchestration harness governs all LLM usage, ensuring bounded autonomy, deterministic tool calls, and state-verified updates rather than unconstrained generation.

The system is a persistent ambient AI layer — a continuously running, multimodal, distributed assistant that integrates into perception and action loops rather than behaving as a discrete chatbot. Its defining property is not model intelligence, but the tightness of its feedback loop between sensing, memory, reasoning, and execution across wearable and home infrastructure.

---

## 12. Offline Optimization

The offline optimization loop runs separately from the real-time runtime but shares the same compute infrastructure. The scheduler supports a dynamic repartition mode where compute allocation shifts toward trace mining, macro discovery, knowledge validation, background optimization, indexing and refinement, and dataset generation for system improvements.

Reserved capacity ensures that safety-critical responsiveness is maintained even during heavy offline processing. The runtime executes; the offline system evolves.

---

## 13. Security and Privacy Considerations

### 13.1 Data Boundaries

The system spans edge devices, home infrastructure, and potentially cloud providers. Data boundaries must be clearly enforced. Raw sensor data, especially biometric data, is processed at the edge whenever possible. Only structured, compressed world-state events are transmitted to the homelab. Cloud providers receive only normalized inference requests through the LLM Runtime, with no access to raw world-state or execution traces.

### 13.2 Biometric Data Handling

EEG, EMG, gaze tracking, and other biometric inputs are highly sensitive personal data. These signals must never leave the edge node in raw form. They are processed locally into derived state estimates — attention level, fatigue index, confirmation signals — which are then fused into the world-state graph. Raw biometric streams are never stored, transmitted, or logged. Derived estimates are ephemeral and tied to the current session context.

### 13.3 Network Security

The PEN maintains a VPN tunnel to the homelab as its primary trusted egress. Multi-WAN aggregation across LTE, Wi-Fi, and Ethernet introduces multiple attack surfaces. Policy-based routing must enforce that all traffic to the homelab traverses the encrypted tunnel. Fallback routing through untrusted networks must be restricted to essential heartbeat signals only, with no world-state or sensor data exposed.

### 13.4 Trust Boundaries Between Nodes

Each node in the distributed system operates with a different trust level. The homelab is the trusted core. The PEN is a semi-trusted mobile node subject to physical compromise. Cloud inference providers are untrusted third parties. The system must enforce mutual authentication between all nodes, encrypt all inter-node communication, and ensure that no single node compromise exposes the full world-state or execution history.

### 13.5 Cryptographic Integrity

Events in the world-state graph are cryptographically hashed and optionally signed. This creates a tamper-evident history where any modification to past events is detectable. The hash chain ensures causal lineage — each event references its parents, making the full provenance chain verifiable.

### 13.6 Graceful Degradation and Offline Safety

When the PEN disconnects from the homelab, it must continue to function safely with local compute only. This means local inference models must have bounded capabilities that do not require homelab coordination. Critical safety chains must remain non-interruptible even in degraded mode. Any action that requires homelab verification must be deferred or denied when offline, never executed speculatively.

### 13.7 Macro Validation and Reversibility

Macros are compiled from execution traces and represent reusable behavior patterns. A malicious or corrupted macro could encode harmful behavior. All macros must pass validation tests before activation. Macros must be fully reversible — the system must be able to decompose any macro back into its constituent primitives and verify its behavior. Macro execution must be auditable through the trace IR, and any macro can be demoted or disabled at any time.

### 13.8 Code Registry Integrity

Code components in the registry are versioned, content-addressed, and cryptographically verifiable. Every component version is immutable once published. The test pipeline ensures that only validated code enters the registry. Cross-graph references link component versions to their test runs in the world-state graph, providing full auditability. Edge nodes verify component integrity through cryptographic hashes before execution.

### 13.9 Channel Security

Channels are trust boundaries between the runtime and external surfaces. Each channel enforces its own authentication, authorization, and rate limiting. Input from any channel is treated as an intent proposal, not a command — the kernel validates and schedules execution regardless of source. Channels on untrusted surfaces (public APIs, third-party messaging platforms) must never receive raw world-state data, only projected and filtered views appropriate to that channel's trust level.

### 13.10 Observability and Audit

All execution is trace-driven. The trace IR provides a complete audit log of every action taken by the system. This audit trail is tamper-evident through cryptographic hashing and accessible for review. Users must be able to inspect the full execution history, understand why actions were taken, and trace any behavior back to its originating intent proposal and sensor inputs.

---

## 14. Glossary

**Execution Chain** — A persistent, prioritized, interruptible cognitive process that may span reasoning steps, tool calls, waiting periods, user interactions, and subchains. The fundamental unit of work in the system.

**Intermediate Representation (IR)** — A canonical, normalized event format that all system components read and write. Enables decoupling between event producers and consumers.

**Trace IR** — The specific intermediate representation used for execution. Contains chain lineage, tool usage, reasoning steps, latency, retries, interruptions, macro usage, and priority context.

**World-State Graph** — The unified conceptual model for all system activity, operationally split into six logical graphs (perception, execution, memory, knowledge, macro, code registry). All follow the same Git-like event-sourced model with cross-references between them.

**Cognitive Runtime Kernel** — The execution engine that manages chains as DAG-based computation graphs, handles scheduling and prioritization, and executes intent proposals using deterministic primitives.

**LLM Runtime** — A standalone adapter layer that standardizes heterogeneous LLM providers into a universal runtime representation. Adapts any model to the normalized IR schema. Completely separate from semantic code components and orchestration primitives.

**Semantic Orchestration Layer** — A minimal set of language-agnostic execution primitives that define what a program means, independent of programming language. Consumed by macros (Tool Services, runtime-only) and code components (Application Services, with transpilation support).

**Tool Services** — Services exposed to macros. Individual tool calls are exposed as service methods. Macros compose these services through semantic orchestration primitives to execute automations. Runtime-only — not transpilable.

**Application Services** — Services exposed to code components. Application-level capabilities — databases, APIs, storage, and native code modules. Code components compose these services through semantic orchestration primitives. Supports transpilation: TS+Effect → Go (nearly 1:1), C++ (runtime layer for Unreal Engine).

**Macro** — An Effect-style function definition that composes Tool Services through semantic orchestration primitives. Discovered from repeated execution patterns, validated through tests, and compiled into reusable execution subgraphs. Runtime-only — not transpilable. Reversible, always expandable to originating events, and never replaces primitives.

**Portable Personal Edge Node (PEN)** — A wearable, battery-powered distributed computing system that extends a user's home AI infrastructure into the physical world as a mobile sensor, network, and execution layer.

**Multimodal Cognitive Interface** — The fusion of partial, noisy evidence from multiple sensor modalities (voice, gaze, EEG, EMG, behavior history) into a unified real-time intent and context graph, with AR as the primary output channel.

**Experience Bubble** — An optional 3D re-renderable spatial memory reconstruction of a visited environment, indexed semantically over time.

**Dynamic Repartition Mode** — A scheduler mode where compute allocation shifts from runtime execution toward offline optimization activities including trace mining, macro discovery, and indexing, while maintaining reserved capacity for safety-critical responsiveness.

**Priority Inheritance** — The mechanism by which urgency from a subchain propagates to its parent chain, ensuring that dependent execution is scheduled correctly.

**Reasoning Processing Unit (RPU)** — A coprocessor pattern that treats an LLM as a specialized reasoning engine rather than an autonomous agent. Receives structured state and returns structured cognitive artifacts.

**Cognitive Cache Hierarchy** — A layered model of cognitive resources from L1 (active context) through L5 (reasoning), where reasoning is the most expensive operation and the runtime's purpose is to convert reasoning into reusable structure.

**Planning-First Execution** — A workflow where plan generation precedes execution, the plan becomes an observable runtime artifact, and execution updates plan state rather than generating arbitrary messages.

**Personality State** — Structured data representing personality traits and behavioral parameters, owned by the runtime and consumed by the RPU as a projection. Enables model replacement, versioning, and evolution.

**Content-Addressed Event** — An event in the world-state graph that is identified by its cryptographic hash, ensuring immutability and tamper-evidence.

**Provenance Chain** — The full causal lineage of an action, traceable from perception processing through world-state version, semantic interpretation, intent proposal, scheduler decision, and tool execution.

**Projection** — A pure transformation function over an event stream that produces durable runtime state. Transforms graph state into graph state (Graph → Projection → Graph). Projections are internal, composable, queryable, and can be chained. Examples include the derivation of execution graphs from event logs, memory graphs from execution graphs, and knowledge graphs from memory graphs. Projections optimize for correctness, persistence, and composition. Classified by determinism level: deterministic (fully replayable, identical output for identical input), probabilistic (LLM-assisted, carries confidence and model version metadata), and hybrid (combines deterministic and probabilistic stages).

**Deterministic Projection** — A projection that produces identical output given identical input. Fully replayable, versioned by logic hash, debugged by replaying input. Examples: perception processing, execution graph derivation.

**Probabilistic Projection** — A projection that may produce varying output across replays due to LLM involvement. Carries confidence scores, versioned by logic hash and model version, debugged by examining evidence trails. Examples: semantic interpretation, intent estimation.

**Hybrid Projection** — A projection that combines deterministic and probabilistic stages. Deterministic stages are fully replayable; probabilistic stages carry confidence and model version metadata. Examples: summarization pipeline, knowledge validation.

**Channel** — A bidirectional boundary between the runtime and the outside world. Transforms graph state into external representation (Graph → Channel → External Surface) and translates external inputs back into events (External Surface → Channel → Event). Channels are consumer-facing interfaces — web UIs, Discord, CLI, telemetry, streaming, search APIs. Channels are leaves in the transformation pipeline; nothing projects further from a channel. They optimize for rendering, filtering, formatting, and aggregation. Channels are replaceable and independent.

**Code Registry** — A homelab-hosted, versioned repository of tested, content-addressed code components written by the model, agents, or user. Applies the same Git-like event-sourced model as the world-state and macro graphs, but as a separate logical graph.

**Code Component** — An Effect-style function definition that composes Application Services through semantic orchestration primitives. A versioned, tested, and reusable unit of code stored in the code registry. Supports transpilation: TS+Effect → Go (nearly 1:1), C++ (runtime layer). Replaces external dependencies for small logic over time.

**Test Pipeline** — A mandatory validation system that every new code component version must pass before adoption. Runs test suites, checks coverage, verifies no regressions, and confirms dependency compatibility.

**Cross-Graph Reference** — A content-addressed link between separate logical graphs (perception, execution, memory, knowledge, macro, code registry). Works like git submodules, enabling full provenance chains across domains while keeping queries simple.

**Perception** — Structured JSON output from signal processing systems (object detection, speech-to-text, attention tracking, movement analysis, cognitive state processing). The first entries in the world-state graph. Raw sensor streams never enter the graph; perception is the structured output of those streams.

**Semantic** — Cross-modal interpretation of perception, produced by LLM-assisted RPU using the JSON in → JSON out pattern. Represents coherent understanding of what is happening. Recorded in the event log as an intermediate transform that feeds into intent estimation.

**Intent** — Convergence of semantic and user signals into action proposals. Consumed by the kernel, which validates, schedules, and executes intent through deterministic primitives. Both user-originated and system-originated intent use the same format.

**Signal-to-Intent Pipeline** — The three-layer process that converts raw sensor streams into actionable intent: perception processing (signal → structure), semantic interpretation (structure → meaning), and intent estimation (meaning → action). All stages are recorded for provenance and model training.

**Perception Graph** — The sub-graph handling structured perception from signal processing systems, environmental facts, user state, and biometrics. Answers "what is happening in the world."

**Execution Graph** — The sub-graph handling chains, tasks, scheduling decisions, tool calls, and RPU invocations. Answers "what is the system doing."

**Memory Graph** — The sub-graph handling episodic experiences, emotional context, narrative memories, personality evolution, and relationship state. Answers "what has been experienced." Retrieved by similarity, temporal proximity, and emotional valence.

**Knowledge Graph** — The sub-graph handling validated facts, semantic knowledge, tested relationships, and crystallized patterns. Answers "what is known to be true." Facts must pass corroboration, contradiction detection, and confidence scoring. Knowledge carries temporal validity, confidence decay, and revision chains — it is durable but not final. Retrieved by semantic query.

**Macro Graph** — The sub-graph handling compiled execution subgraphs. Discovery is passive through sliding window mining. Validation tests against historical traces. Answers "what patterns repeat and whether they are valid."

**Code Registry Graph** — The sub-graph handling versioned code components, tests, and dependencies. Components are actively written by model, agents, or user. A mandatory test pipeline validates before adoption. Answers "what code exists, what version is current, and whether it passes tests."

**Summarization Pipeline** — The downstream pipeline that converts semantic into narrative memories and validated knowledge. Operates after the signal-to-intent pipeline. Makes the six-graph model operationally viable by reducing volume while preserving signal. Knowledge extraction requires corroboration and confidence scoring.

**Narrative Memory** — A consolidated experience indexed for long-term recall. A sequence of semantic compressed into a retrievable memory unit.

**Knowledge Validation** — The process by which proposed facts become validated knowledge. Requires corroboration across independent events, contradiction detection, confidence scoring, and consistency checks. Facts in a "hypothesis" state when contradictory evidence exists. Once sufficient confidence is reached, the fact crystallizes into the knowledge graph with temporal validity, confidence decay, and revision chain support.
