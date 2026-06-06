# Git World-State Graph

> This document expands on the World-State Graph Architecture section (Section 3) of [TECHNICAL_CONCEPT.md](../TECHNICAL_CONCEPT.md).

> **The world-state graph is a Git-like, content-addressed history of cognition. Every perception, reasoning step, tool call, scheduler decision, and state transition is recorded as an immutable event in a causally linked DAG. Current state is not stored separately; it is a projection of the event history.**

### Core Principles

**Events are commits**

- Every Trace IR event is immutable.
- Events reference their causal parents.
- Events are cryptographically hashed and optionally signed.
- History is append-only.

```text
Event A
   ↓
Event B
   ↓
Event C
```

---

**The world-state is a projection**

- Current state is derived from replaying events.
- State is a materialized view, not the source of truth.
- Checkpoints accelerate reconstruction but do not replace history.

```text
History
    ↓
Projection
    ↓
Current World-State
```

---

**Projections transform graph state into graph state**

- Projections are pure functions over event streams.
- They produce durable, queryable, composable knowledge structures.
- They can be chained — output of one becomes input of another.
- They optimize for correctness, persistence, and composition.

```text
Raw Signals
    ↓
Perception Processing
    ↓
Perception
    ↓
Projection
    ↓
Perception Graph
    ↓
Projection
    ↓
Semantic
    ↓
Projection
    ↓
Intent
    ↓
Projection
    ↓
Execution Graph
    ↓
Projection
    ↓
Memory Graph
    ↓
Projection
    ↓
Knowledge Graph
```

---

**Six logical graphs, one conceptual model**

The world-state is operationally split into six logical graphs, each following the same Git-like model:

- **Perception Graph** — structured perception from signal processing (the first graph entries)
- **Execution Graph** — chains, scheduling decisions, tool calls, RPU invocations
- **Memory Graph** — episodic experiences, narrative memories, personality evolution
- **Knowledge Graph** — validated facts with temporal validity, confidence decay, and revision chains
- **Macro Graph** — compiled execution subgraphs, passively discovered through trace mining
- **Code Registry Graph** — versioned code components, actively written and test-validated

Each graph maintains its own projection layer, indexing strategy, and retention policies. Cross-graph references work like git submodules — content-addressed hashes link components across domains while keeping queries simple.

---

**Channels are bidirectional boundaries with the outside world**

- Channels consume projected state and expose it to external surfaces.
- Channels translate external inputs back into events.
- Channels are leaves in the transformation pipeline — nothing projects further from them.
- They are replaceable and independent.

```text
State Graph
    ↓
Channel
    ↓
External Surface (Web, Discord, CLI, API, etc.)
```

```text
External Surface
    ↓
Channel
    ↓
Event
```

---

**The graph is a DAG, not a chain**

- Multiple chains can execute concurrently.
- Reasoning can fork.
- Execution paths can merge.
- Lineage is preserved.

```text
       A
      / \
     B   C
      \ /
       D
```

---

**Execution chains behave like branches**

- Long-running tasks become branches of execution.
- Chains can pause, resume, fork, merge, or terminate.
- Scheduler decisions determine which branches receive resources.

```text
Main
 ├─ Interactive Chain
 ├─ Automation Chain
 └─ Background Chain
```

---

**Macros behave like compiled commit ranges**

- Frequently occurring subgraphs are identified.
- They are validated and compiled.
- They remain reversible.
- A macro can always be expanded back into its originating events.

```text
Events 100-150
      ↓
 Macro #12
      ↓
Expand
      ↓
Events 100-150
```

---

**Provenance is first-class**
Every action should be traceable to:

```text
Perception Processing
        ↓
World-State Version
        ↓
Semantic Interpretation
        ↓
Intent Proposal
        ↓
Scheduler Decision
        ↓
Tool Execution
```

The system can answer:

- What happened?
- Why did it happen?
- What information was available?
- What alternatives existed?
- What chain caused it?

---

**Git, not blockchain**
The goal is not distributed consensus.

The goal is:

- Tamper-evident history
- Causal lineage
- Replayability
- Auditability
- Reproducibility

The architecture is therefore closer to:

```text
Git
+
Event Sourcing
+
Knowledge Graph
+
Distributed Runtime
```

than to a blockchain.

---

### One-Line Architecture Summary

> **The Distributed Cognitive Runtime maintains a Git-like, cryptographically verifiable DAG of world-state events, where cognition, perception, execution, memory, and automation are represented as immutable history, and current reality is a continuously updated projection of that history. Projections transform graph state into graph state; channels expose projected state to external surfaces.**

That statement is the one I'd keep in mind while reviewing future revisions of the docs. If a new subsystem doesn't fit naturally into that model, it's usually a sign that it's introducing a second source of truth or violating one of your core invariants. If a transformation doesn't produce durable runtime knowledge, it's a channel, not a projection. If a channel tries to produce runtime state, it should emit events instead.
