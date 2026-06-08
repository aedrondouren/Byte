# Retrieval Pipeline

> Expands on the Retrieval Model section of [TECHNICAL_CONCEPT.md](../TECHNICAL_CONCEPT.md).
> **Audience:** Technical
> **Prerequisites:** TECHNICAL_CONCEPT.md Sections 1-5, 8; [ENTITIES.md](ENTITIES.md); [CHANNELS.md](CHANNELS.md)
> **Status:** Complete

## One-Line Summary

> **Memory retrieval is the intersection of multiple filters: entity policy, channel policy, memory domain, privacy rules, and task relevance. Only memories satisfying all constraints become part of active reasoning. Knowledge carries dual access levels — factual content propagates globally while contextual metadata remains scoped to originating relationships.**

---

## Core Principle

The runtime distinguishes between factual knowledge and relational context. Retrieval is not a simple similarity search — it is a constrained traversal of the shared graph, filtered by who is asking, through which channel, about what, and with what permissions.

```
Candidate Memory

∩ Internal Entity Policy
∩ External Entity Context
∩ Channel Policy
∩ Memory Domain
∩ Privacy Rules
∩ Task Relevance

= Active Context
```

---

## Retrieval Request

```
Retrieval Request
├── requesting_entity: entity_id
├── via_channel: channel_id
├── task_context:
│   ├── domains: [domain, ...]
│   ├── keywords: [string, ...]
│   └── temporal_context: { window, reference_time }
├── max_results: n
└── urgency: normal | high | critical
```

---

## Filter Chain

### Step 1: Entity Trust Check

Load the requesting entity's trust level and permissions.

```
If entity.trust_level == untrusted:
    -> return empty context (no access)
```

Untrusted entities receive no memory or knowledge context. They may still interact with the system (producing events), but they receive no contextual information in return.

### Step 2: Channel Policy

Load the channel's configuration and compute effective constraints.

```
effective_domains = entity.domain_access ∩ channel.allowed_domains - channel.blocked_domains
effective_privacy_ceiling = min(entity.privacy_ceiling, channel.max_privacy_level)
```

The effective domain set is the intersection of what the entity can access and what the channel allows, minus what the channel explicitly blocks. The effective privacy ceiling is the more restrictive of the entity's ceiling and the channel's ceiling.

### Step 3: Domain and Privacy Filter

Query the memory and knowledge graphs with the effective constraints.

```
Memory Query:
    WHERE domains ∩ effective_domains != empty
    AND privacy_level <= effective_privacy_ceiling

Knowledge Query:
    WHERE domain ∈ effective_domains
    AND privacy_level <= effective_privacy_ceiling
```

This step produces the candidate set — all memories and knowledge entries that pass the domain and privacy filters.

### Step 4: Relationship Context Filter

Apply relationship-aware filtering to prioritize relevant context and enforce dual-access knowledge projection.

```
For each candidate memory:
    If requesting_entity ∈ memory.subjects:
        -> full inclusion (entity is directly involved)
        -> full contextual metadata

For each candidate knowledge entry:
    If requesting_entity == knowledge.learned_from:
        -> full inclusion with full contextual metadata
    Else if requesting_entity has domain access:
        -> factual content included
        -> contextual metadata stripped or anonymized
    Else:
        -> excluded
```

**Dual-access knowledge projection** is the critical mechanism here. Knowledge entries carry two access levels:

```
Knowledge Entry:
├── factual_content: "Event sourcing is useful for execution graphs."
├── metadata:
│   ├── learned_from: entity_admin
│   ├── privacy: private
│   ├── domain: technical
│   └── relationship_context: entity_admin
└── access_levels:
    ├── factual: { domain: technical }          // globally accessible if domain permitted
    └── contextual: { relationship: entity_admin }  // scoped to originating relationship
```

When accessed by the originating entity (Admin), both factual content and contextual metadata are returned. When accessed by a different entity with matching domain permissions, only the factual content is returned — the contextual metadata (who taught this, under what relationship) is stripped or anonymized.

### Step 5: Task Relevance Filter

Apply semantic similarity scoring against the task context and rank results.

```
For each remaining candidate:
    score = semantic_similarity(candidate, task_context)
    score += temporal_proximity(candidate, task_context.temporal_context)
    score += relationship_weight(candidate, requesting_entity)

Sort by score descending
Truncate to max_results
```

### Output: ContextProjection

```
ContextProjection
├── memories: [MemoryEvent, ...]
├── knowledge: [KnowledgeEntry, ...]
├── entities: [Entity, ...]
├── channel: ChannelPolicy
├── active_entity: Entity
└── permission_summary:
    ├── accessible_domains: [domain, ...]
    ├── privacy_ceiling: privacy_level
    └── control_authority: boolean
```

This ContextProjection becomes the `context` field in the RPU request.

---

## RPU Integration

The retrieval pipeline constructs the ContextProjection for each RPU invocation:

```typescript
interface RPURequest {
  function: string;
  objective: string;
  personality: PersonalityState;
  worldState: WorldState;
  context: ContextProjection; // from retrieval pipeline
  taskState?: TaskState;
  previousArtifacts?: Artifact[];
}

interface ContextProjection {
  memories: MemoryEvent[];
  knowledge: KnowledgeEntry[];
  entities: Entity[];
  channel: ChannelPolicy;
  activeEntity: Entity;
  permissionSummary: {
    accessibleDomains: string[];
    privacyCeiling: string;
    controlAuthority: boolean;
  };
}
```

---

## Dual-Access Knowledge: Detailed Behavior

### Access Matrix

| Accessor                         | Factual Content | Contextual Metadata                        |
| -------------------------------- | --------------- | ------------------------------------------ |
| Admin (originating relationship) | Full            | Full (learned_from, relationship, privacy) |
| admin_delegate (domain match)    | Full            | Stripped or anonymized                     |
| trusted_user (domain match)      | Full            | Stripped                                   |
| sub_user (domain match)          | Full            | Stripped                                   |
| Any entity (no domain match)     | None            | None                                       |
| Untrusted entity                 | None            | None                                       |

### Example

```
Knowledge Entry K1:
    factual_content: "User prefers concise responses."
    metadata:
        learned_from: entity_admin
        privacy: private
        domain: interaction
        relationship_context: entity_admin

Access by Admin:
    -> factual_content: "User prefers concise responses."
    -> metadata: { learned_from: entity_admin, privacy: private, ... }

Access by sub_user Bob (domain: interaction = read):
    -> factual_content: "User prefers concise responses."
    -> metadata: { domain: interaction }  // learned_from stripped

Access by untrusted entity:
    -> nothing
```

This prevents accidental leakage of contextual information while allowing useful abstractions to propagate.

---

## Permission-Based Invalidation

When entity permissions change, derived artifacts are affected:

### Knowledge Invalidation

```
Entity demoted or permissions reduced:
    ↓
Identify all knowledge entries where learned_from == demoted_entity
    ↓
Apply accelerated confidence decay
    ↓
Restrict access based on new permission profile
    ↓
If new entity (with different permissions) produces same factual patterns:
    -> new knowledge entries created with new provenance
    -> old entries decay naturally
```

### Macro Invalidation

```
Entity demoted or permissions reduced:
    ↓
Identify all macros derived from demoted_entity's execution
    ↓
Demote macros (status: source_permission_changed)
    ↓
If entity is re-elevated and produces same patterns:
    -> new macros proposed with updated provenance
```

Memories and events are never modified — they are the immutable shared substrate. Only derived artifacts (knowledge, macros) are invalidated.

---

## Performance Considerations

### Indexing Strategy

The retrieval pipeline relies on efficient indexing across multiple dimensions:

| Index                | Purpose                       | Complexity                            |
| -------------------- | ----------------------------- | ------------------------------------- |
| Domain index         | Filter by domain membership   | O(log n)                              |
| Privacy index        | Filter by privacy level       | O(log n)                              |
| Entity-subject index | Filter by involved entities   | O(1) hash lookup                      |
| Semantic index       | Task relevance scoring        | O(log n) approximate nearest neighbor |
| Temporal index       | Temporal proximity scoring    | O(log n)                              |
| Relationship index   | Relationship-weighted ranking | O(1) hash lookup                      |

### Caching

Channels are responsible for their own caching of world-state projections. The retrieval pipeline produces fresh context on each RPU invocation. Caching, memoization, and performance optimization are handled at the channel/implementation layer, not as an architectural subsystem.

---

## Cross-References

- [TECHNICAL_CONCEPT.md](../TECHNICAL_CONCEPT.md) — Memory graph, knowledge graph, RPU contract
- [ENTITIES.md](ENTITIES.md) — Entity permissions, trust levels
- [CHANNELS.md](CHANNELS.md) — Channel policies, entity binding
- [RPU.md](RPU.md) — RPU request/response schema
- [GRAPH.md](GRAPH.md) — Indexing strategy, query complexity
