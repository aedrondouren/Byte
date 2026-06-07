# Design Decisions

> Rationale behind key architectural choices in the B.Y.T.E. architecture.
> **Audience:** Technical
> **Prerequisites:** TECHNICAL_CONCEPT.md (full)
> **Status:** Complete

This document explains why the architecture makes the choices it does. Each decision includes the alternatives considered and the reasoning for the chosen approach.

---

## Why "Git, Not Blockchain"

**Decision:** The world-state graph uses a Git-like model (content-addressed, immutable, DAG-structured) rather than a blockchain model.

**Alternatives considered:**

- Blockchain with consensus (Bitcoin/Ethereum model)
- Append-only log with centralized authority (Kafka model)
- Mutable database with audit log (traditional model)

**Reasoning:**

- The goal is tamper-evident history, causal lineage, replayability, auditability, and reproducibility — not distributed consensus. There is a single trusted authority (the homelab), so consensus mechanisms are unnecessary overhead.
- Git's DAG model naturally represents causal relationships between events. Each event references its parents, creating a verifiable chain of causality.
- Content-addressing (identifying events by their hash) ensures immutability without requiring a consensus protocol.
- The blockchain analogy breaks down on branching/merging semantics. Git's branch model (lightweight, mergeable, with conflict detection) is closer to the execution chain model than blockchain's linear chain model.

**Where the analogy breaks down:**

- Git is designed for human-authored code with infrequent merges. The world-state graph is machine-authored with frequent concurrent branches. Merge semantics differ.
- Git's conflict resolution is manual. The world-state graph needs automated conflict resolution (deferred to Phase 6).
- Git stores full file contents. The world-state graph stores structured events with projections.

---

## Why Six Graphs vs. One Graph with Tags

**Decision:** The world-state is conceptually split into six logical graphs (perception, execution, memory, knowledge, macro, registry) with a single physical event store.

**Alternatives considered:**

- One graph with type tags on all events
- Six separate physical databases
- One graph with separate indexes per domain

**Reasoning:**

- Conceptual separation keeps queries simple and indexing efficient. A query for "all memories relevant to the current situation" should not scan perception events.
- Physical unification (single event store) ensures that cross-graph references are content-addressed hashes within the same namespace, simplifying provenance chains.
- Each graph maintains its own indexing strategy and retention policies. Perception events may be retained for days; knowledge entries may be retained indefinitely.
- The "six graphs, one store" model (TECHNICAL_CONCEPT.md Section 1.5) provides the best balance of conceptual clarity and implementation simplicity.

---

## Why TypeScript + Effect as the Primitive Vocabulary

**Decision:** The semantic orchestration layer uses TypeScript + Effect as the reference implementation and common primitive vocabulary.

**Alternatives considered:**

- Rust (strong type safety, but different concurrency model)
- Go (simple concurrency, but lacks Effect's error handling model)
- Custom DSL (full control, but high implementation cost)
- Python (ecosystem, but lacks type safety for semantic guarantees)

**Reasoning:**

- Effect.ts provides a comprehensive, well-tested set of semantic primitives (run, fork, parallel, race, scope, acquire/release, etc.) that map closely to the orchestration layer's requirements.
- TypeScript's type system provides compile-time guarantees about primitive composition.
- Effect's layer/context system provides the dependency injection model needed for Tool Services and Application Services.
- TypeScript + Effect transpiles nearly 1:1 to Go (REGISTRY.md), enabling cross-language composition.
- The choice is about semantics, not implementation. The primitives are language-agnostic; TypeScript + Effect is the reference.

---

## Why Event-Sourced vs. State-Based

**Decision:** The world-state is event-sourced (append-only event log, state as projection) rather than state-based (mutable database with current state).

**Alternatives considered:**

- Mutable database with audit log
- Event sourcing with periodic snapshots as primary state
- CRDT-based eventual consistency

**Reasoning:**

- Event sourcing provides full provenance. Every action is traceable to its origin. This is essential for debugging, auditability, and system improvement.
- State as projection means the system can always rebuild current state from history. Checkpoints accelerate reconstruction but do not replace history.
- The append-only model simplifies concurrency. There are no write conflicts because events are never modified.
- The trade-off is storage growth and query complexity. The summarization pipeline (TECHNICAL_CONCEPT.md Section 8) addresses storage by compressing events into memories and knowledge. Indexing strategies (GRAPH.md) address query complexity.

---

## Why RPU as Coprocessor vs. Autonomous Agent

**Decision:** The LLM is a reasoning coprocessor (RPU) invoked by the kernel with structured contracts, not an autonomous agent that controls the system.

**Alternatives considered:**

- Autonomous agent (LLM controls execution, tool use, and memory)
- ReAct-style agent (LLM interleaves reasoning and action)
- Tool-use agent (LLM calls tools through a defined interface)

**Reasoning:**

- Separation of concerns: the kernel manages execution, scheduling, and safety. The RPU performs transformations that genuinely require reasoning. This prevents the LLM from making irreversible decisions without validation.
- Model independence: the RPU contract is model-agnostic. Swapping models does not change system behavior.
- Auditability: every RPU invocation is a structured event with input, output, and metadata. This is easier to audit than an autonomous agent's free-form reasoning.
- Cost control: the kernel decides when to invoke the RPU and with what context. This prevents unnecessary inference calls.
- The "AI proposes intent; the kernel executes deterministically" invariant (TECHNICAL_CONCEPT.md Section 1.4) ensures that the system remains safe even if the LLM produces incorrect or malicious output.

---

## Why Projection vs. Channel Distinction

**Decision:** The runtime distinguishes between projections (graph → graph transformations) and channels (graph → external surface transformations).

**Alternatives considered:**

- Single transformation model (no distinction)
- Pipeline model (all transformations are sequential)
- Event-driven model (all transformations are event handlers)

**Reasoning:**

- Projections build runtime knowledge; channels consume it. This distinction prevents channels from creating a second source of truth.
- Projections are durable; channels come and go. The same projection graph can serve any number of channels without conceptual changes.
- The decision rules are actionable: "If a transformation doesn't produce durable runtime knowledge, it's a channel, not a projection." This prevents architectural drift.
- Channels are leaves in the transformation pipeline. Nothing projects further from a channel. This prevents circular dependencies between external surfaces and internal state.

---

## Why Defer Known Gaps

Several gaps are acknowledged but not resolved (KNOWN_GAPS.md). The deferral rationale:

- **Macro discovery:** The core thesis holds through memory and knowledge compression even if macro discovery fails. Macro discovery is an optimization, not a foundation. Deferred until Phases 1–4 validate the thesis.
- **Knowledge extraction rules:** Will emerge from implementation and testing rather than theoretical design. The validation pipeline (corroboration, contradiction detection, confidence scoring) provides a framework; the specific rules need empirical tuning.
- **Distributed conflict resolution:** The architecture is single-node with edge-cloud separation. Multi-node conflict resolution is a Phase 6 problem. CRDTs, vector clocks, or merge policies will be evaluated when the need arises.
- **EEG and experimental modalities:** Not required for the core thesis. Included to explore the concept's boundaries, not as foundational requirements.

---

**Related:** [KNOWN_GAPS.md](KNOWN_GAPS.md) for the specific gaps that remain unresolved. [TECHNICAL_CONCEPT.md](../TECHNICAL_CONCEPT.md#14-core-invariants) for the core invariants that drive these decisions.
