# Reference

## Glossary and Terminology

This document defines all terms used across the B.Y.T.E. architecture specification.

---

### Core Concepts

**Execution Chain** — A persistent, prioritized, interruptible process that may span reasoning steps, tool calls, waiting periods, user interactions, and subchains.

**Intermediate Representation (IR)** — A canonical, normalized event format that all system components read and write.

**Trace IR** — The specific intermediate representation used for execution. Contains chain lineage, tool usage, reasoning steps, latency, retries, interruptions, macro usage, and priority context.

**World-State Graph** — The unified conceptual model for all system activity, operationally split into six logical graphs (perception, execution, memory, knowledge, macro, code registry).

### Runtime Components

**Cognitive Runtime Kernel** — The execution engine that manages chains as DAG-based computation graphs, handles scheduling and prioritization, and executes intent proposals using deterministic primitives.

**LLM Runtime** — A standalone adapter layer that standardizes heterogeneous LLM providers into a universal runtime representation.

**Reasoning Processing Unit (RPU)** — A coprocessor pattern that treats an LLM as a specialized reasoning engine rather than an autonomous agent. Receives structured state and returns structured results.

**Semantic Orchestration Layer** — A minimal set of language-agnostic execution primitives that define what a program means, independent of programming language.

### Services and Composition

**Tool Services** — Services exposed to macros. Individual tool calls as service methods. Runtime-only, not transpilable.

**Application Services** — Services exposed to code components. Application-level capabilities — databases, APIs, storage, native code modules. Supports transpilation: TS+Effect → Go (nearly 1:1), C++ (runtime layer).

### Derived Graphs

**Macro** — An Effect-style function definition that composes Tool Services through semantic orchestration primitives. Discovered from repeated execution patterns, validated through tests, compiled into reusable execution subgraphs. Runtime-only, reversible, never replaces primitives.

**Code Registry** — A homelab-hosted, versioned repository of tested, content-addressed code components. Used by the model, agents, and users when writing software together — not executed by the runtime.

**Code Component** — An Effect-style function definition that composes Application Services through semantic orchestration primitives. A versioned, tested, reusable unit of code stored in the code registry and drawn from when writing software — not executed by the runtime.

**Test Pipeline** — A mandatory validation system that every new code component version must pass before adoption.

### Edge and Interface

**Portable Personal Edge Node (PEN)** — A wearable, battery-powered distributed computing system that extends a user's home AI infrastructure into the physical world as a mobile sensor, network, and execution layer.

**Multimodal Cognitive Interface** — The fusion of partial, noisy evidence from multiple sensor modalities (voice, gaze, EEG, EMG, behavior history) into a unified real-time intent and context graph, with AR as the primary output channel.

**Experience Bubble** — An optional 3D re-renderable spatial memory reconstruction of a visited environment, indexed semantically over time.

### Execution Model

**Dynamic Repartition Mode** — A scheduler mode where compute allocation shifts from runtime execution toward offline optimization activities while maintaining reserved capacity for safety-critical responsiveness.

**Priority Inheritance** — The mechanism by which urgency from a subchain propagates to its parent chain, ensuring that dependent execution is scheduled correctly.

**Cognitive Cache Hierarchy** — A layered model of cognitive resources from L1 (active context) through L5 (reasoning), where reasoning is the most expensive operation.

**Planning-First Execution** — A workflow where plan generation precedes execution, the plan becomes an observable runtime artifact, and execution updates plan state rather than generating arbitrary messages.

**Personality State** — Structured data representing personality traits and behavioral parameters, owned by the runtime and consumed by the RPU as a projection.

### Automation

**Temporal Intent** — Scheduled intent generated from knowledge entries carrying temporal patterns. Proactive rather than reactive. Fires automatically when confidence is high; user denial feeds back into knowledge validation.

**Temporal Intent Generator** — A kernel component that monitors the knowledge graph for active temporal patterns (recurring schedules and one-shot triggers). Produces intent when conditions are met, with provenance back to the originating knowledge entry.

### Data Model

**Content-Addressed Event** — An event in the world-state graph that is identified by its cryptographic hash, ensuring immutability and tamper-evidence.

**Provenance Chain** — The full causal lineage of an action, traceable from perception processing through world-state version, situation model generation, intent proposal, scheduler decision, and tool execution.

**Projection** — A pure transformation function over an event stream that produces durable runtime state. Transforms graph state into graph state. Classified by determinism level: deterministic, probabilistic, and hybrid.

**Channel** — A bidirectional boundary between the runtime and the outside world. Transforms graph state into external representation and translates external inputs back into events. Leaves in the transformation pipeline.

**Cross-Graph Reference** — A content-addressed link between separate logical graphs. Works like git submodules, enabling full provenance chains across domains.

### Pipeline Stages

**Perception** — Structured JSON output from signal processing systems. The first entries in the world-state graph.

**Situation Model** — Cross-modal interpretation of perception, produced by LLM-assisted RPU using the JSON in → JSON out pattern. An intermediate transform that feeds into intent estimation.

**Intent** — Convergence of situation model, user signals, and temporally-scheduled knowledge into action proposals. Consumed by the kernel, which validates, schedules, and executes through deterministic primitives.

**Signal-to-Intent Pipeline** — The three-layer process that converts raw sensor streams into actionable intent: perception processing, situation model generation, and intent estimation.

**Summarization Pipeline** — The downstream pipeline that converts situation model into narrative memories and validated knowledge. Makes the six-graph model operationally viable by reducing volume while preserving signal.

**Narrative Memory** — A consolidated experience indexed for long-term recall. A sequence of situation model compressed into a retrievable memory unit.

**Knowledge Validation** — The process by which proposed facts become validated knowledge. Requires corroboration across independent events, contradiction detection, confidence scoring, and consistency checks.
