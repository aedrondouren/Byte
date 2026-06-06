# Distributed Cognitive Runtime (Unified Definition)

> This document is a condensed reference summary of [TECHNICAL_CONCEPT.md](../TECHNICAL_CONCEPT.md).

## One-line definition

> A distributed, event-sourced cognitive runtime where all perception, reasoning, tool execution, and media output are unified into a single canonical world-state graph, continuously optimized via deterministic execution tracing and reversible compilation of behavior into reusable semantic macros.

---

# 🧱 Core Concept

You are building a system that replaces:

- "agent loops"
- "prompt chains"
- "tool-using chatbots"

with:

> a **persistent execution runtime over a shared world-state graph**

Everything (LLM, sensors, tools, streaming, automation) becomes a _projection or transformation of that graph_.

---

# 🧩 1. World-State Graph (Single Source of Truth)

All system activity becomes events in one canonical structure:

- perception from signal processing (object detection, speech-to-text, attention tracking)
- tool calls (filesystem, web, APIs)
- reasoning steps (LLM outputs as structured IR)
- execution chains (tasks, automation flows)
- environmental state (home + mobile + remote)

### Key property:

> Nothing is "input vs output" — everything is an event in the same system.

> Raw sensor streams never enter the graph. Camera feeds, audio, gaze, IMU, EEG are piped to specialized processing systems. The first graph entries are structured perception.

---

# 🔄 2. Signal-to-Intent Pipeline

Raw signals are converted into actionable intent through three layers:

### Perception Processing (Signal → Structure)

Specialized systems (YOLO, Whisper, attention trackers) convert raw sensor streams into structured perception. Deterministic, model-based, not LLM-based. Runs on edge and homelab.

### Semantic Interpretation (Structure → Meaning)

Cross-modal interpretation of perception using the RPU pattern: JSON in → JSON out. LLM-assisted, runs on homelab. Produces coherent understanding of what is happening.

### Intent Estimation (Meaning → Action)

Converges semantic and user signals into unified intent. Flows through three stages:

**Intent** — state change request (what is desired)
**Proposal** — executable candidate (how to do it)
**Commit** — kernel-approved execution (authorization to proceed)

Both user-originated and system-originated intent use the same three-stage flow.

### Key property:

> All stages are recorded for provenance and model training. Perception is preserved so downstream interpretation can be reprocessed with better models.

---

# ⚙️ 3. Execution Kernel (Cognitive Runtime)

This is the execution kernel.

It manages:

- execution chains (DAG-based, not linear)
- scheduling + prioritization
- interruption + resumption
- long-lived workflows
- resource arbitration
- tool execution lifecycle

### Important shift:

> The LLM does not "run the system" — it proposes structured intent into the kernel.

### Key boundary:

> The scheduler, execution workers, and cache system operate over the graph, not inside it. They are ephemeral runtime infrastructure, not persisted state.

---

# 🧠 4. Reasoning Processing Unit

The RPU treats the LLM as a specialized reasoning coprocessor. JSON in → JSON out. Fixed schemas. No free-form text.

The same RPU pattern governs both reasoning and semantic interpretation — the model only ever sees structured data.

### Key principle:

> Intelligence is not the amount of reasoning performed. Intelligence is the ability to accumulate structure so that less reasoning is required in the future.

---

# 🔌 5. Semantic Orchestration Layer

You define a:

> minimal semantic orchestration API shared across languages

Examples:

- run / fork / parallel / race
- scope / acquire / release
- fail / recover / mapError
- stream / pipe / sink
- provide / layer / context

### Two service ecosystems

**Tool Services** — consumed by macros. Individual tool calls as service methods. Runtime-only, not transpilable.

**Application Services** — consumed by code components. Application-level capabilities including native modules. Supports transpilation: TS+Effect → Go (nearly 1:1), C++ (runtime layer).

The two ecosystems never intersect.

This is NOT a DSL — it is a **portable execution meaning layer**.

---

# 📊 6. Macro System

Effect-style function definitions that compose Tool Services. Passively discovered through trace mining. Runtime-only — no transpilation. Reversible — always expandable to originating events.

---

# 📊 7. Code Registry

Effect-style function definitions that compose Application Services. Actively written, must pass mandatory test pipeline. Supports transpilation: TS+Effect → Go (nearly 1:1), C++ (runtime layer for Unreal Engine).

Macros use Tool Services only. Code components use Application Services only. The two ecosystems never intersect.

---

# 📝 8. Event Summarization and Indexing

Downstream of the signal-to-intent pipeline. Converts accumulated experience into progressively compact representations:

- semantic → narrative memories → validated knowledge

Knowledge carries temporal validity, confidence decay, and revision chains — it is durable but not final.

---

# 🌍 9. Edge Architecture

### Home system:

- fixed cameras
- environmental mapping
- persistent spatial memory
- homelab cognitive core

### Wearable "backpack" node (PEN):

- mobile sensors
- multi-uplink networking
- live environmental injection
- extends world-state beyond home boundary

### Key idea:

> The system does not "observe the world" — it continuously _extends its world-state graph through mobile perception nodes_.

---

# 👁️ 10. Multimodal Cognitive Interface

Inputs are not commands — they are probabilistic signals:

- gaze → attention + selection
- voice → explicit intent
- EMG/blink → confirmation signals
- EEG → state modulation (fatigue/stress/load)
- behavior history → priors

### Key principle:

> Interaction = probabilistic intent estimation, not deterministic control.

AR interface acts as the primary output channel.

---

# 🧭 11. Offline Optimization Loop

Separate from runtime:

- macro mining
- trace compression
- clustering of execution patterns
- refinement of semantic primitives
- dataset generation for improvements

### Key idea:

> Runtime executes. Offline system evolves.

---

# 🔀 12. Projections and Channels

The runtime distinguishes between two categories of transformation:

### Projections

Transform graph state into graph state. Internal, durable, composable.

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
```

Classified by determinism level:

- **Deterministic** — fully replayable, identical output for identical input
- **Probabilistic** — LLM-assisted, carries confidence and model version metadata
- **Hybrid** — combines deterministic and probabilistic stages

Optimize for: correctness, queryability, persistence, composition.

### Channels

Transform graph state into external representation. Bidirectional boundaries with the outside world.

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
Perception Graph → Streaming Channel
```

Optimize for: rendering, filtering, sorting, formatting, aggregation.

### Key distinction:

> Projections build runtime knowledge. Channels consume it.

Projections are durable; channels come and go. The same projection graph serves any number of channels without conceptual changes.

---

# 🧭 Final Unified Model

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

The full transformation flow:

```
Raw Signals
    ↓
Perception Processing → Perception
    ↓
Semantic Interpretation → Semantic
    ↓
Intent Estimation → Intent
    ↓
Projection → Execution Graph
    ↓
Projection → Memory Graph
    ↓
Projection → Knowledge Graph
    ↓
Channel → External Surface (Web, Discord, CLI, API, etc.)
```

---

# 🧠 Core philosophical invariant

Across everything:

> The system is not a chatbot, not an agent, and not a framework — it is a persistent, distributed, event-sourced execution substrate where cognition, perception, and action are unified through a canonical world-state graph.
