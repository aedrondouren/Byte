# Distributed Cognitive Runtime

## Technical Concept Document

---

## Related Documents

- [GRAPH.md](core/GRAPH.md) — Git-like world-state graph model (immutable events, DAG, provenance)
- [RPU.md](core/RPU.md) — Reasoning Processing Unit coprocessor pattern
- [EDGE.md](core/EDGE.md) — Portable Personal Edge Node architecture
- [BCI.md](core/BCI.md) — Multimodal cognitive interface and BCI
- [CODE.md](core/CODE.md) — Semantic orchestration layer, cross-language adapters, and code registry
- [CORE.md](core/CORE.md) — Condensed reference summary

---

## 1. Overview

A distributed, event-sourced cognitive operating system where all perception, reasoning, tool execution, and media output are unified into a single canonical world-state graph, continuously optimized via deterministic execution tracing and reversible compilation of behavior into reusable semantic macros.

This system replaces agent loops, prompt chains, and tool-using chatbots with a persistent execution runtime over a shared world-state graph. Everything — LLM inference, sensor inputs, tool calls, streaming, automation — becomes a projection or transformation of that graph.

The world-state graph is a Git-like, content-addressed history of cognition. Every perception, reasoning step, tool call, scheduler decision, and state transition is recorded as an immutable event in a causally linked DAG. Current state is not stored separately; it is a projection of the event history.

The core philosophical invariant: the system is not a chatbot, not an agent, and not a framework. It is a persistent, distributed, event-sourced execution substrate where cognition, perception, and action are unified through a canonical world-state graph.

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

## 3. World-State Graph

The world-state graph is the single source of truth for the entire system. It is a canonical event graph where every piece of system activity is recorded as a structured event with normalized arguments, timestamps, and dependency edges.

IR stands for Intermediate Representation — a canonical, normalized event format that all system components read and write. The Trace IR is the specific intermediate representation used for execution events. It decouples event producers (LLMs, sensors, tools) from event consumers (scheduler, macro system, observability pipeline), enabling each component to operate independently without knowledge of the others' internal formats.

The world-state graph contains:

- Sensor inputs from cameras, gaze trackers, audio, EEG, EMG, and IMU
- Tool calls to filesystems, web APIs, MCP servers, and system commands
- Reasoning steps from LLM outputs as structured IR
- Execution chains representing tasks, automation flows, and subchains
- Environmental state from home systems, mobile nodes, and remote infrastructure

Nothing is treated as input versus output. Everything is an event in the same system.

### 3.1 Events as Immutable Commits

Every Trace IR event is immutable. Events reference their causal parents, are cryptographically hashed and optionally signed, and the history is append-only. This creates a tamper-evident record of all system activity.

### 3.2 State as Projection

Current state is derived from replaying events. State is a materialized view, not the source of truth. Checkpoints accelerate reconstruction but do not replace history. The system can always rebuild current world-state from the full event log.

### 3.3 DAG Structure

The graph is a DAG, not a linear chain. Multiple chains execute concurrently. Reasoning can fork. Execution paths can merge. Lineage is preserved across all branches. Execution chains behave like branches — long-running tasks become branches of execution that can pause, resume, fork, merge, or terminate. The scheduler determines which branches receive resources.

### 3.4 Provenance

Provenance is first-class. Every action is traceable to its origin: sensor evidence leads to world-state version, which leads to LLM intent proposal, which leads to scheduler decision, which leads to tool execution. The system can answer what happened, why it happened, what information was available, what alternatives existed, and which chain caused it.

### 3.5 Git, Not Blockchain

The goal is not distributed consensus. The goal is tamper-evident history, causal lineage, replayability, auditability, and reproducibility. The architecture is closer to Git plus Event Sourcing plus Knowledge Graph plus Distributed Runtime than to a blockchain.

---

## 4. Cognitive Runtime Kernel

The kernel is the operating system equivalent. It manages execution chains as DAG-based computation graphs rather than linear sequences. It handles scheduling, prioritization, interruption and resumption, long-lived workflows, resource arbitration, and tool execution lifecycle.

The kernel receives structured intent proposals from the LLM and other sources, validates them against current world-state, schedules them according to priority and resource availability, and executes them using the semantic orchestration primitives defined in the compatibility layer.

Key responsibilities:

- Execution chain lifecycle management with states including active, waiting, parked, resumed, and escalated
- Priority-based weighted scheduling across critical, interactive, automation, and background tiers
- Dynamic repartition of compute resources between runtime execution and offline optimization
- Priority inheritance across full execution chains so that subchain urgency propagates correctly
- Hard non-interruptibility for critical safety and security chains
- Interruptible inference for non-critical chains only

### 4.1 RPU Integration

The kernel invokes the Reasoning Processing Unit as a specialized coprocessor, not as an autonomous agent. The RPU receives structured requests containing world-state projections, personality state, context, and task definitions. It returns structured cognitive artifacts — plans, summaries, state update proposals, and action proposals — rather than raw conversational text.

### 4.2 Planning-First Execution

Long-running tasks follow a planning-first workflow. The user request triggers an acknowledgement, then plan generation by the RPU, then execution, then a recap, then the conversation response. The plan becomes an observable runtime artifact. Execution updates the plan state rather than generating arbitrary conversational messages. This creates deterministic visibility into system progress.

### 4.3 Recap-Based Observability

The runtime maintains execution state independently from the conversation. Users may request current status, task progress, plan state, or execution recap. The response is generated from runtime projections rather than reconstructed from conversation history. This avoids using conversational messages as the source of truth.

---

## 5. Reasoning Processing Unit

The RPU treats a Large Language Model as a specialized reasoning coprocessor rather than as a complete autonomous agent. Instead of embedding memory, planning, identity, world state, execution management, and communication into a single prompt, these concerns are externalized into dedicated runtime systems. The model becomes a deterministic cognitive transformation engine operating over structured state.

The objective is not to maximize model intelligence, but to minimize the amount of reasoning required to produce intelligent behavior. Intelligence is the ability to accumulate structure so that less reasoning is required in the future.

### 5.1 RPU Responsibilities

The RPU is responsible for: interpretation, reasoning, synthesis, planning, summarization, reflection, communication generation, and cognitive transformations.

The RPU is not responsible for: memory persistence, event storage, scheduling, state tracking, personality storage, tool orchestration, world modeling, or execution management. These functions belong to the runtime.

### 5.2 Cognitive Contract

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

### 5.3 Personality as State

Personality is stored as structured data rather than prompt text. As an illustration, personality might be represented as dimensional traits:

```json
{
  "curiosity": 0.9,
  "humor": 0.4,
  "technicalDepth": 0.95,
  "directness": 0.8
}
```

The runtime owns personality. The RPU consumes personality projections. This allows model replacement, personality versioning, personality evolution, and consistent behavior across backends.

### 5.4 Cognitive Cache Hierarchy

The architecture operates as a hierarchy of increasingly expensive cognitive operations:

- L1 — Active Context
- L2 — Structured State
- L3 — World Model
- L4 — Event History
- L5 — Reasoning (RPU)

Reasoning is the most expensive resource. The purpose of the runtime is to convert reasoning into reusable structure and prevent unnecessary recomputation.

### 5.5 Guiding Question

The central design question of the RPU architecture is: "What can be removed from the reasoning loop?" Any responsibility that can be represented as deterministic state, stored structure, or runtime infrastructure should be externalized from the model. The RPU should only perform transformations that genuinely require reasoning.

### 5.6 Model Independence

The RPU abstraction enables model interchangeability. The runtime defines state, contracts, events, personality, memory, and scheduling. The model only implements reasoning functions. This allows multiple model classes to participate in the same cognitive runtime.

### 5.7 Event-Sourced Cognition

All runtime actions generate events. The event log becomes the canonical source of truth. The conversation is merely one projection of runtime state.

---

## 6. Provider Translation Layer

A standalone adapter service that standardizes heterogeneous inference providers into a universal runtime representation. This layer is reusable across multiple systems and is not tied to any specific scheduler or automation logic.

Responsibilities include normalizing streaming inference outputs, translating tool-call formats across providers, preserving reasoning and structured outputs when available, abstracting OpenAI-compatible and native APIs, and decoupling providers from downstream systems.

The translation layer ensures that the cognitive runtime kernel never needs to know which specific LLM provider is backing a given inference request. All providers emit the same IR format.

---

## 7. Semantic Orchestration Layer

Instead of transpiling code or cloning runtimes across languages, the system defines a minimal set of Effect-like orchestration primitives that are semantically equivalent across all runtime backends. These primitives define what a program means, independent of any programming language.

Core primitives include run, fork, parallel, race, scope, acquire, release, fail, recover, mapError, stream, pipe, sink, provide, layer, and context. These are not a domain-specific language. They are a portable execution meaning layer.

Above the primitives sits a service-based composition layer where external capabilities — databases, APIs, storage, AI tools, engine APIs — are expressed as injectable services with consistent contracts. Services define the leaves of execution, while the core workflow logic remains pure and portable.

---

## 8. Edge Perception Layer

The edge perception layer extends the world-state graph spatially through mobile and fixed sensor nodes. It consists of two physical components.

The home system provides fixed cameras, environmental mapping, persistent spatial memory, and the homelab cognitive core. It is the anchor point for all long-term reasoning and memory.

The wearable backpack node — the Portable Personal Edge Node, or PEN — provides mobile sensors, multi-uplink networking, live environmental injection, and extends the world-state beyond the home boundary. It is a battery-powered distributed computing system consisting of a wearable compute core connected to heterogeneous sensor devices including phones, glasses, cameras, optional depth sensors, and biometric inputs.

The system does not observe the world. It continuously extends its world-state graph through mobile perception nodes.

---

## 9. Multimodal Intent Layer

Human inputs are not commands. They are probabilistic signals fused into a unified intent and context graph.

Gaze provides attention and selection signals. Voice provides explicit intent. EMG and blink signals provide confirmation. EEG provides state modulation including fatigue, stress, and cognitive load. Behavior history provides priors for intent estimation. Personality state modulates how the system interprets and responds to fused intent.

Interaction is probabilistic intent estimation, not deterministic control. No single modality is sufficient on its own. The system fuses partial, noisy evidence from multiple modalities to estimate what the user intends, and the kernel validates and executes only when confidence thresholds are met.

---

## 10. Macro System

Repeated execution patterns are detected through sliding window mining over event streams, subgraph matching in trace graphs, and normalization of tool calls. The macro system operates as a reversible execution compression pipeline.

The process follows four steps. A macro is proposed, assisted by LLM analysis of trace patterns. It is validated through tests against historical execution data. Once validated, it is compiled into a reusable execution unit. It can be demoted if it falls out of use or if underlying primitives change.

Macros are compiled execution subgraphs, not behavior overrides. They are an optimization layer for execution efficiency and reliability. They never replace primitives, and they remain fully reversible.

### 10.1 Macros as Compiled Commit Ranges

Frequently occurring subgraphs are identified, validated, and compiled. A macro can always be expanded back into its originating events. This guarantees that no behavior is ever hidden behind macro abstraction — the full provenance chain remains accessible.

---

## 11. Code Registry

The code registry applies the same Git-like, event-sourced model used for world-state and macros to the domain of versioned code components. It is a homelab-hosted repository where every piece of code — written by the model, by autonomous agents, or by the user — is tested, versioned, and stored as a reusable, content-addressed artifact.

Over time, this builds an in-house library that replaces external dependencies for small logic. Core packages (frameworks, bundlers, compilers) remain as dependencies, but business logic, common patterns, and utility code are internal, tested, and versioned within the registry.

### 11.1 Relationship to Macros

Macros and code components are two applications of the same underlying pattern: repeated behavior is captured, validated, and made reusable. The distinction is in discovery and validation.

Macros are passively discovered through trace mining and validated against historical execution data. They compress execution patterns.

Code components are actively written and must pass a mandatory test pipeline before adoption. They compress implementation patterns.

Both follow the "history is more important than current state" principle. Both are versioned, reversible, and traceable. But they live in separate logical graphs to keep queries simple and maintenance manageable.

### 11.2 Separate Graphs, Unified Model

The system maintains three logical graphs under the same conceptual model:

The world-state graph handles perception, execution, sensor data, and tool calls. Events happen passively. The kernel validates intent. Query patterns answer what happened, why, and when.

The macro graph handles compiled execution subgraphs. Discovery is passive through sliding window mining. Validation tests against historical traces. Query patterns answer what patterns repeat and whether they are valid.

The code registry graph handles versioned code components, tests, and dependencies. Components are actively written by model, agents, or user. A mandatory test pipeline validates before adoption. Query patterns answer what code exists, what version is current, and whether it passes tests.

Each graph maintains its own projection layer for queries. The raw event log is only for reconstruction, audit, and debugging. Cross-graph references work like git submodules — content-addressed hashes link code components to macro bodies, macro executions to world-state events, and test runs to both the code registry and the world-state graph.

### 11.3 Mandatory Test Pipeline

Unlike macros, which are passively discovered and validated against historical data, code components must actively pass a test pipeline before they are adopted into the registry. Every new version of a code component must:

- Pass its full test suite
- Maintain or improve coverage thresholds
- Not introduce regressions in dependent components
- Be compatible with declared dependencies

The test pipeline runs in the homelab's trusted environment. Only components that pass all checks are published to the registry. Failed versions are recorded in the event log but never become available for use.

### 11.4 Homelab as Trusted Source

The code registry lives on the homelab as the trusted source of truth. Edge nodes pull from it when needed — initially rarely, but as the system grows, edge devices including small laptop setups and the PEN may pull tested components for local execution.

The registry is versioned and content-addressed. Edge nodes pin to specific versions and can verify integrity through cryptographic hashes. No untested or unversioned code is ever distributed.

### 11.5 Goal: In-House Logic, Minimal External Dependencies

The long-term objective is to replace external dependencies for small logic with tested, versioned, in-house components. Most application code consists of common patterns — data transformation, validation, formatting, state management, error handling — that can be captured, tested, and reused.

Core infrastructure packages (frameworks, bundlers, compilers, database drivers) will always be external. But the logic that glues them together becomes internal, auditable, and continuously improving through reuse.

---

## 12. Offline Optimization

The offline optimization loop runs separately from the real-time runtime but shares the same compute infrastructure. The scheduler supports a dynamic repartition mode where compute allocation shifts toward trace mining, macro discovery, background optimization, indexing and refinement, and dataset generation for system improvements.

Reserved capacity ensures that safety-critical responsiveness is maintained even during heavy offline processing. The runtime executes; the offline system evolves.

---

## 13. Media and Streaming Projection

Streaming is not a standalone feature. It is a projection of the world-state graph. Camera feeds are subgraphs of perception. Overlays are state annotations. Clipping is trace selection. Replay is graph reconstruction.

The homelab functions as a live production and orchestration renderer of the world-state graph, enabling multi-camera IRL streaming, contextual overlays, and reconstructable spatial memory visualization.

---

## 14. Execution Model

The system runs on a unified abstraction: an execution chain is a persistent, prioritized, interruptible cognitive process.

Each chain persists across time, survives waiting and interruption, is scheduled dynamically, can spawn subchains, can be resumed after suspension, and is governed by priority and resource quotas.

Chains are not sequences of prompts or tool calls. They are long-lived chains of computation that may include reasoning steps, tool execution, waiting for external events, user interaction, retries and recovery flows, and nested subchains.

The scheduler operates on execution opportunities, not queue draining. It continuously allocates inference and tool capacity across competing chains based on priority, resource availability, and current world-state.

### 13.1 Chain Lifecycle States

- **Active** — currently executing
- **Waiting** — paused for an external event or signal
- **Parked** — suspended due to resource constraints, lower priority
- **Resumed** — restored from waiting or parked state
- **Escalated** — priority elevated due to dependency or user intervention

### 13.2 Priority Tiers

- **Critical** — safety, security, system integrity. Non-interruptible.
- **Interactive** — user-facing responses. High priority, interruptible by critical.
- **Automation** — background workflows, tool chains. Interruptible by critical and interactive.
- **Background** — indexing, optimization, non-urgent tasks. Interruptible by all higher tiers.

### 13.3 Branch Semantics

Execution chains behave like branches in a version control system. They can fork when reasoning diverges, merge when execution paths converge, and terminate when complete. The scheduler manages resource allocation across all active branches, ensuring that critical chains receive priority while background work proceeds when capacity allows.

---

## 15. Cross-Language Runtime Adapters

The semantic orchestration layer is implemented through multiple runtime adapters, each mapping the same execution semantics onto native primitives of the host environment.

The TypeScript runtime is built on Effect for web and backend orchestration, using its native task, scope, resource, and concurrency model.

The Go runtime maps the same semantics onto goroutines, contexts, channels, and native concurrency primitives, providing equivalent behavior with Go's execution model.

The C++ runtime targets Unreal Engine integration, wrapping engine tasking and lifecycle systems to provide the same semantic primitives within a game engine context.

Each adapter does not translate code literally. It implements the same execution semantics using native primitives of the host environment. The result is a system where workflow logic is portable and language-agnostic, runtime behavior is native and optimized per platform, and differences between environments are isolated to service implementations and runtime adapters rather than business logic.

---

## 16. Edge Architecture

The Portable Personal Edge Node operates through five coordinated layers.

**Edge Perception Layer** handles real-time multi-camera egocentric capture, depth sensing and IMU tracking, audio capture and translation, local object detection and scene parsing, and produces structured, compressed world-state events rather than raw media streams.

**Local Deterministic Compute Layer** runs low-latency inference models, sensor fusion and SLAM, networking and routing, policy enforcement, AR rendering and interaction logic, and ensures the system remains stable even when offline.

**Remote Cognitive Layer** on the homelab provides high-capacity reasoning and planning, long-term memory and trip reconstruction, scene re-synthesis through 3D splats or neural reconstruction, streaming production and content orchestration, and identity, personality, and dialogue continuity.

**Network Orchestration Layer** manages multi-WAN aggregation across LTE, Wi-Fi, and Ethernet, maintains a VPN tunnel to home as the primary trusted egress, enforces policy-based routing balancing latency versus trust versus cost, and keeps an always-on heartbeat maintaining global state continuity.

**Memory and World Model** maintains a continuous heartbeat telemetry stream, logs events rather than raw media, reconstructs spatial representations of visited environments, semantically indexes experiences over time, and optionally supports 3D re-renderable experience bubbles.

The system functions as a personal mobile operating environment for perception and cognition, enabling augmented navigation and awareness, live contextual translation and information overlay, distributed multi-camera IRL streaming, reconstructable spatial memory of lived environments, and continuous synchronization with a home-based intelligence core.

---

## 17. Multimodal Cognitive Interface

The system is a world-state engine that continuously updates a structured representation of the environment, user attention, and internal state. Inputs come from multiple modalities — voice, gaze tracking, IMU and head pose, cameras in RGB, depth, and IR, EEG providing low-bandwidth cognitive and context signals, and EMG or micro-gestures for confirmation.

Each sensor contributes partial, noisy evidence that is fused into a unified real-time intent and context graph.

The hierarchical control stack separates concerns by latency. Low-latency wearable or backpack compute handles perception, signal filtering, and immediate interaction logic. The homelab system provides heavier reasoning, long-term memory, planning, and tool execution. A custom orchestration harness governs all LLM usage, ensuring bounded autonomy, deterministic tool calls, and state-verified updates rather than unconstrained generation.

The AR interface acts as the primary output layer, turning the internal world model into spatially grounded overlays, notifications, and adaptive UI elements.

EEG and other biosignals do not act as direct command channels. They are contextual modulation signals that adjust system behavior based on attention, fatigue, stress, or engagement, improving the timing and relevance of interactions.

The system is a persistent ambient AI layer — a continuously running, multimodal, distributed assistant that integrates into perception and action loops rather than behaving as a discrete chatbot. Its defining property is not model intelligence, but the tightness of its feedback loop between sensing, memory, reasoning, and execution across wearable and home infrastructure.

---

## 18. Security and Privacy Considerations

### Data Boundaries

The system spans edge devices, home infrastructure, and potentially cloud providers. Data boundaries must be clearly enforced. Raw sensor data, especially biometric data, is processed at the edge whenever possible. Only structured, compressed world-state events are transmitted to the homelab. Cloud providers receive only normalized inference requests through the provider translation layer, with no access to raw world-state or execution traces.

### Biometric Data Handling

EEG, EMG, gaze tracking, and other biometric inputs are highly sensitive personal data. These signals must never leave the edge node in raw form. They are processed locally into derived state estimates — attention level, fatigue index, confirmation signals — which are then fused into the world-state graph. Raw biometric streams are never stored, transmitted, or logged. Derived estimates are ephemeral and tied to the current session context.

### Network Security

The PEN maintains a VPN tunnel to the homelab as its primary trusted egress. Multi-WAN aggregation across LTE, Wi-Fi, and Ethernet introduces multiple attack surfaces. Policy-based routing must enforce that all traffic to the homelab traverses the encrypted tunnel. Fallback routing through untrusted networks must be restricted to essential heartbeat signals only, with no world-state or sensor data exposed.

### Trust Boundaries Between Nodes

Each node in the distributed system operates with a different trust level. The homelab is the trusted core. The PEN is a semi-trusted mobile node subject to physical compromise. Cloud inference providers are untrusted third parties. The system must enforce mutual authentication between all nodes, encrypt all inter-node communication, and ensure that no single node compromise exposes the full world-state or execution history.

### Cryptographic Integrity

Events in the world-state graph are cryptographically hashed and optionally signed. This creates a tamper-evident history where any modification to past events is detectable. The hash chain ensures causal lineage — each event references its parents, making the full provenance chain verifiable.

### Graceful Degradation and Offline Safety

When the PEN disconnects from the homelab, it must continue to function safely with local compute only. This means local inference models must have bounded capabilities that do not require homelab coordination. Critical safety chains must remain non-interruptible even in degraded mode. Any action that requires homelab verification must be deferred or denied when offline, never executed speculatively.

### Macro Validation and Reversibility

Macros are compiled from execution traces and represent reusable behavior patterns. A malicious or corrupted macro could encode harmful behavior. All macros must pass validation tests before activation. Macros must be fully reversible — the system must be able to decompose any macro back into its constituent primitives and verify its behavior. Macro execution must be auditable through the trace IR, and any macro can be demoted or disabled at any time.

### Code Registry Integrity

Code components in the registry are versioned, content-addressed, and cryptographically verifiable. Every component version is immutable once published. The test pipeline ensures that only validated code enters the registry. Cross-graph references link component versions to their test runs in the world-state graph, providing full auditability. Edge nodes verify component integrity through cryptographic hashes before execution.

### Observability and Audit

All execution is trace-driven. The trace IR provides a complete audit log of every action taken by the system. This audit trail is tamper-evident through cryptographic hashing and accessible for review. Users must be able to inspect the full execution history, understand why actions were taken, and trace any behavior back to its originating intent proposal and sensor inputs.

---

## 19. Glossary

**Execution Chain** — A persistent, prioritized, interruptible cognitive process that may span reasoning steps, tool calls, waiting periods, user interactions, and subchains. The fundamental unit of work in the system.

**Intermediate Representation (IR)** — A canonical, normalized event format that all system components read and write. Enables decoupling between event producers and consumers.

**Trace IR** — The specific intermediate representation used for execution events. Contains chain lineage, tool usage, reasoning steps, latency, retries, interruptions, macro usage, and priority context.

**World-State Graph** — The single source of truth for the entire system. A canonical event graph where every piece of system activity is recorded as a structured event with normalized arguments, timestamps, and dependency edges.

**Cognitive Runtime Kernel** — The execution engine that manages chains as DAG-based computation graphs, handles scheduling and prioritization, and executes intent proposals using deterministic primitives.

**Provider Translation Layer** — A standalone adapter service that standardizes heterogeneous inference providers into a universal runtime representation, decoupling providers from downstream systems.

**Semantic Orchestration Layer** — A minimal set of language-agnostic execution primitives that define what a program means, independent of programming language. Implemented through runtime adapters for TypeScript, Go, and C++.

**Macro** — A compiled execution subgraph discovered from repeated execution patterns, validated through tests, and used to compress frequent reasoning and tool patterns into reliable reusable flows. Reversible, always expandable to originating events, and never replaces primitives.

**Portable Personal Edge Node (PEN)** — A wearable, battery-powered distributed computing system that extends a user's home AI infrastructure into the physical world as a mobile sensor, network, and execution layer.

**Multimodal Intent Layer** — The fusion of partial, noisy evidence from multiple sensor modalities (voice, gaze, EEG, EMG, behavior history) into a unified real-time intent and context graph.

**Experience Bubble** — An optional 3D re-renderable spatial memory reconstruction of a visited environment, indexed semantically over time.

**Dynamic Repartition Mode** — A scheduler mode where compute allocation shifts from runtime execution toward offline optimization activities including trace mining, macro discovery, and indexing, while maintaining reserved capacity for safety-critical responsiveness.

**Priority Inheritance** — The mechanism by which urgency from a subchain propagates to its parent chain, ensuring that dependent execution is scheduled correctly.

**Reasoning Processing Unit (RPU)** — A coprocessor pattern that treats an LLM as a specialized reasoning engine rather than an autonomous agent. Receives structured state and returns structured cognitive artifacts.

**Cognitive Cache Hierarchy** — A layered model of cognitive resources from L1 (active context) through L5 (reasoning), where reasoning is the most expensive operation and the runtime's purpose is to convert reasoning into reusable structure.

**Planning-First Execution** — A workflow where plan generation precedes execution, the plan becomes an observable runtime artifact, and execution updates plan state rather than generating arbitrary messages.

**Personality State** — Structured data representing personality traits and behavioral parameters, owned by the runtime and consumed by the RPU as a projection. Enables model replacement, versioning, and evolution.

**Content-Addressed Event** — An event in the world-state graph that is identified by its cryptographic hash, ensuring immutability and tamper-evidence.

**Provenance Chain** — The full causal lineage of an action, traceable from sensor evidence through world-state version, LLM intent proposal, scheduler decision, and tool execution.

**Code Registry** — A homelab-hosted, versioned repository of tested, content-addressed code components written by the model, agents, or user. Applies the same Git-like event-sourced model as the world-state and macro graphs, but as a separate logical graph.

**Code Component** — A versioned, tested, and reusable unit of code stored in the code registry. Replaces external dependencies for small logic over time.

**Test Pipeline** — A mandatory validation system that every new code component version must pass before adoption. Runs test suites, checks coverage, verifies no regressions, and confirms dependency compatibility.

**Cross-Graph Reference** — A content-addressed link between separate logical graphs (world-state, macro, code registry). Works like git submodules, enabling full provenance chains across domains while keeping queries simple.
