# Git World-State Graph

> Expands on the World-State Graph Architecture section of [TECHNICAL_CONCEPT.md](../TECHNICAL_CONCEPT.md#4-world-state-graph-architecture).

> **The world-state graph is a Git-like, content-addressed history of cognition. Every perception, reasoning step, tool call, scheduler decision, and state transition is recorded as an immutable event in a causally linked DAG. Current state is not stored separately; it is a projection of the event history.**

### Events are commits

```text
Event A
    ↓
Event B
    ↓
Event C
```

Every event is immutable, references its causal parents, is cryptographically hashed and optionally signed. History is append-only.

### State is a projection

```text
History
    ↓
Projection
    ↓
Current World-State
```

Current state is derived from replaying events. Checkpoints accelerate reconstruction but do not replace history.

### The full projection pipeline

```text
Raw Signals
    ↓
Perception Processing
    ↓
Perception
    ↓
Perception Graph
    ↓
Situation Model Generation
    ↓
Situation Model
    ↓
Intent Estimation
    ↓
Intent
    ↓
Execution Graph
    ↓
Memory Graph
    ↓
Knowledge Graph

Skill (pre-authored, executed via RPU)
    ↓
Execution Graph (skill execution traces)
    ↓
Macro Graph (skill-derived, with provenance)
    ↓
Knowledge Graph (skill-derived, with provenance)
```

Each arrow is a projection — a pure function over an event stream. Skill-derived projections carry provenance metadata linking derived artifacts to their source skill and version.

### Seven logical graphs, two stores

The seven graphs are split across two data stores reflecting two fundamentally different data models.

**World-State Event Store** — append-only, content-addressed events. State is derived by projecting event history.

```text
Perception Graph  ── what happened       (event store)
Execution Graph   ── what was done       (event store)
Memory Graph      ── what was experienced (event store)
Knowledge Graph   ── what is known       (event store)
```

**Artifact Version Store** — content-addressed, semantically versioned, lifecycle-managed. State is the latest version per artifact.

```text
Macro Graph       ── compiled patterns   (artifact store)
Code Registry     ── tested components   (artifact store)
Skill Registry    ── authored behaviors  (artifact store)
```

Cross-store references use content hashes in events pointing to artifact ID+version. No unified index is needed.

### Channels are bidirectional boundaries

```text
State Graph → Channel → External Surface (Web, Discord, CLI, API)
External Surface → Channel → Event
```

Channels are leaves in the transformation pipeline. Nothing projects further from a channel.

### The graph is a DAG

```text
      A
     / \
    B   C
     \ /
      D
```

Multiple chains execute concurrently. Reasoning can fork. Execution paths can merge. Lineage is preserved.

### Execution chains behave like branches

```text
Main
 ├─ Interactive Chain
 ├─ Automation Chain
 └─ Background Chain
```

Long-running tasks become branches of execution that can pause, resume, fork, merge, or terminate.

### Macros are compiled commit ranges

```text
Events 100-150
    ↓
Macro #12
    ↓
Expand
    ↓
Events 100-150
```

A macro can always be expanded back into its originating events. No behavior is hidden behind abstraction.

### Provenance is first-class

```text
Perception Processing
    ↓
World-State Version
    ↓
Situation Model Generation
    ↓
Intent Proposal
    ↓
Scheduler Decision
    ↓
Tool Execution
```

The system can answer: what happened, why, what information was available, what alternatives existed, what chain caused it.

### Git, not blockchain

```text
Git
+
Event Sourcing
+
Knowledge Graph
+
Distributed Runtime
```

The goal is tamper-evident history, causal lineage, replayability, auditability, and reproducibility — not distributed consensus.

---

### One-Line Architecture Summary

> **The Distributed Cognitive Runtime maintains a Git-like, cryptographically verifiable DAG of world-state events, where cognition, perception, execution, memory, and automation are represented as immutable history, and current reality is a continuously updated projection of that history. Projections transform graph state into graph state; channels expose projected state to external surfaces.**

### Query Complexity and Cross-Graph Resolution

**Cross-graph reference resolution.** Within the world-state event store, graphs reference each other through content-addressed hashes. Resolving a cross-graph reference requires:

1. **Hash lookup** in the target graph's index (O(1) with hash table, O(log n) with sorted index).
2. **Event retrieval** from the world-state event store (O(1) with content-addressed storage).
3. **Recursive resolution** if the target event itself contains cross-graph references.

The maximum resolution depth is bounded by the projection chain length (typically 2–4 hops: perception → situation model → intent → execution → memory).

**Cross-store reference resolution.** When a world-state event references an artifact (e.g., `macro_used: macro_weather_check:v3.1.0`):

1. **Parse artifact reference** from the event (O(1) — structured field).
2. **Artifact lookup** in the artifact version store by ID+version (O(1) with hash table).
3. **Artifact retrieval** from the artifact store (O(1) with content-addressed storage).

When an artifact references world-state events for provenance (e.g., `provenance: { source_events: [evt_abc, ...] }`):

1. **Parse event hashes** from the artifact's provenance field (O(1)).
2. **Event retrieval** from the world-state event store (O(1) per hash).

No unified index is needed. Each store maintains its own indexing.

**Query cost analysis for common patterns:**

| Query Pattern                                      | Complexity       | Index Used                         | Notes                                                                     |
| -------------------------------------------------- | ---------------- | ---------------------------------- | ------------------------------------------------------------------------- |
| "All events in time window [t1, t2]"               | O(log n + k)     | Time-sorted index                  | k = matching events                                                       |
| "All chains with status X"                         | O(log n + k)     | Status index                       | Execution graph only                                                      |
| "Memories similar to context C"                    | O(n \* d)        | Semantic index (approximate)       | d = embedding dimension; approximate nearest neighbor reduces to O(log n) |
| "Knowledge about topic T"                          | O(log n + k)     | Topic index                        | Knowledge graph only                                                      |
| "Cross-graph: execution triggered by perception P" | O(1 + log n + k) | Cross-reference hash + chain index | Single hash lookup + chain traversal                                      |
| "Full provenance chain for event E"                | O(d)             | Parent reference traversal         | d = chain depth (typically < 10)                                          |
| "All macros derived from skill S"                  | O(1 + k)         | Provenance index                   | k = derived macros                                                        |
| "All knowledge derived from skill S version V"     | O(1 + k)         | Provenance index + confidence      | k = knowledge entries                                                     |
| "Current version of skill S"                       | O(log n)         | Version index                      | Semantic version lookup                                                   |
| "Current version of artifact A"                    | O(1)             | Artifact index                     | Latest version lookup                                                     |
| "All versions of artifact A"                       | O(k)             | Version index                      | k = version count                                                         |
| "Artifacts referencing event E"                    | O(log n)         | Reverse reference index            | k = referencing artifacts                                                 |

**Indexing strategy performance:**

_World-State Event Store:_

- **Perception graph:** Time-sorted index + spatial index (quadtree for location-based queries). Query cost: O(log n) for time range, O(log n) for spatial range.
- **Execution graph:** Chain ID index (hash table) + status index (sorted) + tool type index (hash table). Query cost: O(1) for chain lookup, O(log n) for status/tool queries.
- **Memory graph:** Semantic similarity index (vector database, approximate nearest neighbor). Query cost: O(log n) with HNSW or similar algorithm.
- **Knowledge graph:** Topic index (hash table) + confidence index (sorted) + validity window index (interval tree). Query cost: O(1) for topic lookup, O(log n) for confidence/validity queries.

_Artifact Version Store:_

- **Macro graph:** Pattern index (hash table by normalized signature) + usage index (sorted by last-used time) + provenance index (hash table by source skill ID and version) + version index (sorted by semantic version). Query cost: O(1) for pattern lookup, O(log n) for usage queries, O(1) for provenance lookup, O(1) for version lookup.
- **Skill registry:** Capability index (hash table by skill name/ID) + version index (sorted by semantic version) + tool access index (hash table by tool name). Query cost: O(1) for skill lookup, O(log n) for version queries, O(1) for tool access queries.
- **Code registry:** Capability index (hash table by component capability) + version index (sorted by semantic version) + dependency index (hash table by dependency). Query cost: O(1) for capability lookup, O(log n) for version queries, O(1) for dependency queries.

**Retention policy impact on query cost.** Events outside the retention window are archived to cold storage. Queries that span hot and cold storage incur additional latency for cold storage retrieval. The summarization pipeline ensures that frequently-queried data (knowledge, recent memories) remains in hot storage, while raw perception events are archived after the retention period.

Artifacts in the version store are never archived to cold storage as long as any reference points to them. An artifact version remains accessible until all references to it are removed (macro demoted, skill uninstalled, code component deprecated with no dependents). This differs from event retention: artifacts are kept by reference, not by time window.

### Design Review Heuristics

Keep this statement in mind while reviewing future revisions. If a new subsystem doesn't fit naturally into that model, it's introducing a second source of truth or violating a core invariant.

- If a transformation doesn't produce durable runtime knowledge, it's a channel, not a projection.
- If a channel tries to produce runtime state, it should emit events instead.
