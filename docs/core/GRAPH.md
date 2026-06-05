# Git World-State Graph

> This document expands on the World-State Graph section of [TECHNICAL_CONCEPT.md](../TECHNICAL_CONCEPT.md).

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
Sensor Evidence
       ↓
World-State Version
       ↓
LLM Intent Proposal
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

> **The Distributed Cognitive Runtime maintains a Git-like, cryptographically verifiable DAG of world-state events, where cognition, perception, execution, memory, and automation are represented as immutable history, and current reality is a continuously updated projection of that history.**

That statement is the one I'd keep in mind while reviewing future revisions of the docs. If a new subsystem doesn't fit naturally into that model, it's usually a sign that it's introducing a second source of truth or violating one of your core invariants.
