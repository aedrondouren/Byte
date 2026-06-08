# Entity Model

> Expands on the Entity Model section of [TECHNICAL_CONCEPT.md](../TECHNICAL_CONCEPT.md).
> **Audience:** Technical
> **Prerequisites:** TECHNICAL_CONCEPT.md Sections 1-5, 8
> **Status:** Complete

## One-Line Summary

> **Entities are lightweight graph nodes with identity, relationships, and permissions — not memory containers. All memory lives in the shared graph with metadata describing context. Entities define retrieval policies over that shared graph.**

---

## Entity Store Strategy

Entity definitions and entity state live in different stores, reflecting their different natures:

**Entity definitions** (identity, trust level, permissions, relationships) are stored as versioned artifacts in the **Entity Graph** within the artifact version store. Entity definitions change through discrete actions — admin elevation, identity merge, permission changes, relationship updates — and are managed as versioned, content-addressed artifacts with lifecycle states.

**Entity state** (last seen, interaction count, derived preferences, behavioral patterns) is a **projection** over the world-state event stream. It is computed from all events involving the entity and is not stored as an artifact. State can be cached for performance but is always derivable from history.

```
Entity Definition (Artifact Store)          Entity State (Projection)
├── id, type, trust_level                   ├── last_seen: timestamp
├── identity, relationships                 ├── interaction_count: number
├── permissions, lifecycle_state            ├── derived_preferences: [...]
                                            └── ...  // all derived from events
```

See [Entity Schema](#entity-schema) below for the complete schema.

**Why not derive everything from events?** Entity definitions are queried on every retrieval request, every channel binding, and every permission check. Re-deriving identity, trust, and permissions from the full event stream on every access would be prohibitively expensive. The artifact store provides efficient O(1) definition lookup while preserving the "state as projection" invariant for derived state.

**Known vs. Unknown Entities.** An entity becomes "known" once it has an entry in the Entity Graph — regardless of trust level. A known entity carries accumulated identity identifiers from multiple channels, relationship history, and associated knowledge/memories in the world-state graphs. If a known entity is demoted (e.g., from `trusted_user` back to `untrusted`), the entity definition remains in the Entity Graph with reduced permissions, but all accumulated identity, relationship context, and historical knowledge persists. The system retains understanding of who this entity is even without runtime access. An "unknown" entity is a raw channel identifier with no Entity Graph entry — detected but not yet promoted to an entity definition.

**The complete entity view at runtime** combines three sources:

```
Entity Definition (Entity Graph, artifact store)
    +
Entity State (projection from event stream)
    +
Channel Binding (entity_bindings in Channel definition)
    =
Complete Entity/Session Context (runtime)
```

This composition means that the same entity definition can produce different runtime contexts when bound to different channels, and that entity state is always current (projected from the latest events) while the definition is versioned and auditable.

---

## Core Principle

The runtime does not distinguish between "agent memory" and "user memory." Instead, it maintains a single immutable history and a unified knowledge graph from which multiple projections are derived. Entities are graph nodes with relationships and metadata, not containers for isolated memories.

```
Entity
├── Identity
├── Type
├── Relationships
├── State (projection of history)
└── Permissions (retrieval policy)
```

---

## Entity Schema

### Entity Definition (stored in Entity Graph, artifact version store)

```
Entity Definition
├── id: content-addressed hash
├── type: internal | external
├── internal_type: technical_assistant | companion | stream_host | automation_agent  // internal only
├── trust_level: admin | admin_delegate | trusted_user | sub_user | untrusted
├── identity:
│   ├── primary: { channel_id, identifier }
│   └── merged: [ { channel_id, identifier, merged_at, merged_by } ]
├── relationships: [ { target_entity_id, relation_type, strength, since } ]
├── permissions: { ... }
└── lifecycle_state: discovered | pending_review | active | suspended | merged | revoked
```

### Entity State (projected from world-state event stream)

```
Entity State
├── last_seen: timestamp
├── interaction_count: number
├── derived_preferences: [ ... ]
├── behavioral_patterns: [ ... ]
└── ...  // all fields derived from events involving this entity
```

The entity definition is a versioned artifact — it changes only through discrete actions and each change produces a new version. Entity state is a continuous projection — it evolves with every event involving the entity and is computed on demand or cached for performance.

---

## Trust Levels

| Level            | Description                                           | Authority                                                           |
| ---------------- | ----------------------------------------------------- | ------------------------------------------------------------------- |
| `admin`          | Primary Admin — single, bootstrap entity, irrevocable | Full system authority. Root of trust.                               |
| `admin_delegate` | Admin user (Discord-style server admin)               | Can manage channels, entities, permissions. Cannot supersede Admin. |
| `trusted_user`   | Trusted participant                                   | Expanded domain/tool/sensor access. May have limited delegation.    |
| `sub_user`       | Elevated external entity                              | Limited, specific permissions granted by Admin or admin_delegate.   |
| `untrusted`      | Default for all new entities                          | No access to any domain, tool, sensor, or control signal.           |

### Admin Bootstrap

The Admin entity exists from system start, pre-linked to the primary webUI session:

```
System Bootstrap:
    ↓
Create Entity: admin
├── trust_level: admin
├── primary_identity: { channel: "webui_primary", id: "bootstrap" }
├── permissions: { all: allowed }
└── status: active
    ↓
Admin uses system, gradually associates:
    ├── Name, preferences (manual input or learned patterns)
    ├── Additional identity merges (Discord, voice, etc.)
    └── General information (location, schedule, etc.)
```

No approval workflow is needed for Admin — it is the system's root of trust.

### Admin Ceiling Invariant

No permission delegation chain can produce effective permissions equal to or greater than Admin. Admin is always the ceiling. This is enforced by construction:

- `admin_delegate` permissions are always a strict subset of Admin's
- `trusted_user` permissions are always a strict subset of `admin_delegate`'s
- `sub_user` permissions are always a strict subset of `trusted_user`'s
- The Admin trust level is singular and irrevocable

---

## Internal Entities

Internal entities represent different operational projections of the runtime. They do not own separate memory stores. Instead, they define retrieval policies over the shared graph.

Examples:

- **Technical Assistant** — focused on technical domains, restricted from personal/private
- **Companion** — focused on interaction and relationship domains
- **Stream Host** — focused on public-facing technical and project domains
- **Automation Agent** — focused on technical and project domains, with tool/sensor permissions for automated tasks

Internal entities do not require Admin elevation — they are part of the system definition.

```
Internal Entity: Technical Assistant

Accessible domains:
    technical
    project
    research

Restricted domains:
    personal
    private
    credentials

Tool permissions:
    read_code
    run_tests
    deploy

Sensor permissions:
    camera_read
    mic_read

Control channel: denied
```

An internal entity specifies:

- Accessible memory domains
- Accessible privacy levels
- Allowed channels
- Tool permissions
- Sensor permissions (read vs. control)
- Control channel authority

---

## External Entities

External entities represent observed participants in the environment.

Examples:

- Administrator (additional identities merged to Admin entity)
- Family member
- Guest user
- Pet
- Smart device

External entities are constructed from history and graph relationships. They start as **untrusted** with no access and require explicit elevation.

### External Entity Lifecycle

```
Unknown Identifier (raw channel identifier, no Entity Graph entry)
    ↓
Entity Discovery (new participant detected from channel event)
    ↓
Create Entity Definition v1 (Entity Graph, lifecycle_state: discovered)
├── trust_level: untrusted
├── identity: { primary: { channel_id, identifier } }
└── permissions: { none }
    ↓
Entity Definition v1.1 (lifecycle_state: pending_review)  // flagged for Admin review
    ↓
Admin or admin_delegate reviews entity
    ↓
Elevate to sub_user -> Entity Definition v2 (lifecycle_state: active)
├── trust_level: sub_user
├── permissions: { specific grants }
└── provenance: { elevated_by: admin_action_event, from_version: v1.1 }
    ↓
Further elevation to trusted_user -> Entity Definition v3
├── trust_level: trusted_user
├── permissions: { expanded grants }
└── provenance: { elevated_by: admin_action_event, from_version: v2 }
    ↓
Possible elevation to admin_delegate -> Entity Definition v4
├── trust_level: admin_delegate
├── permissions: { administrative grants }
└── provenance: { elevated_by: admin_action_event, from_version: v3 }
    ↓
Possible demotion -> Entity Definition v5
├── trust_level: untrusted (or sub_user)
├── permissions: { reduced or none }
└── provenance: { demoted_by: admin_action_event, from_version: v4 }
    └── NOTE: All accumulated identity, relationships, and historical knowledge persist
```

**Demotion preserves knowledge.** When a known entity is demoted, the entity definition is updated with reduced permissions but is not deleted. All identity identifiers, relationship history, and associated knowledge/memories in the world-state graphs persist. The system retains understanding of who this entity is — it simply cannot access runtime resources. If the entity is later re-elevated, the accumulated context is immediately available.

**Key invariant:** Trust is never automatic. No entity becomes trusted through behavior alone. Trust is explicitly granted by a higher-trust entity.

### Operational Models

The runtime does not attempt to create complete psychological models. Instead, it maintains operational models derived from observed interactions.

```
History
    ↓
Memory Extraction
    ↓
Knowledge Graph
    ↓
Entity Projection
```

External entity projections may contain:

- Communication preferences
- Roles and permissions
- Relationship state
- Interaction history
- Contextual knowledge

---

## Entity State as Projection

An entity's "state" is not stored separately — it is a projection of all events and memories involving that entity:

```
Entity State = Projection of:
├── All events where entity was subject or participant
├── All memories where entity is a subject
├── All knowledge where entity is the provenance source
├── All relationships with other entities
└── Interaction patterns (frequency, timing, preferences)
```

This is consistent with BYTE's existing "state as projection" invariant (TECHNICAL_CONCEPT.md, Section 4.4).

---

## Identity and Merging

### Identity Structure

Each entity has a primary identity (the channel and identifier through which it was first discovered) and may have merged identities (additional channel identifiers confirmed to be the same entity):

```
Entity: Admin
├── primary_identity: { channel: "webui_primary", id: "bootstrap" }
├── merged_identities: [
│       { channel: "discord", id: "@admin_user", merged_at: "...", merged_by: "admin" },
│       { channel: "voice", id: "voice_print_abc", merged_at: "...", merged_by: "admin" }]
└── trust_level: admin
```

### Identity Resolution

When an event arrives from a channel with an unknown identifier:

```
Event from channel with unknown identifier
    ↓
Query: Does identifier match any known entity?
    ↓
If YES -> attach to existing entity, update state
If NO -> create untrusted entity, flag for review
    ↓
Check for potential identity matches:
    ├── Same name across channels
    ├── Same voice print / biometric signature
    ├── Same behavioral patterns
    └── Same network/context
    ↓
If potential match -> present merge suggestion
```

### Merge Permissions

| Actor                                    | Can Merge                                        |
| ---------------------------------------- | ------------------------------------------------ |
| Admin                                    | Any identities                                   |
| admin_delegate (with `merge_identities`) | Any identities within their permission scope     |
| trusted_user (with `merge_identities`)   | Own identities only (self-merge across channels) |
| sub_user                                 | Cannot merge                                     |

All merges are logged as events. Admin is notified of all merges, even delegated ones. Admin can reverse merges.

### Identity Merging as a DAG Operation

Identity merging is a DAG operation in the Entity Graph. When two entity definitions are determined to represent the same participant, they are merged into a single successor definition that preserves both as ancestors:

```
Entity Def v1: { channel: "discord", id: "@admin_user" }     Entity Def v2: { channel: "voice", id: "voice_print_abc" }
                                \                            /
                                 \                          /
                        Entity Def v3: merged identity
                        ├── identity:
                        │   ├── primary: { channel: "discord", id: "@admin_user" }
                        │   └── merged: [ { channel: "voice", id: "voice_print_abc", ... } ]
                        ├── trust_level: admin (inherited from highest ancestor)
                        ├── permissions: union of ancestor permissions
                        └── provenance: { merged_from: [v1, v2], merged_by: admin, merged_at: ... }
```

The merged entity definition is a new version with two parents in the DAG. This preserves the provenance of each identity source and enables reversal if needed. All events referencing the original entity definitions continue to reference them; new events reference the merged definition.

**Merge constraints:**

- Only entities of the same type can be merged (external with external, internal with internal)
- The merged entity inherits the highest trust level of its ancestors
- Permissions are the union of ancestor permissions, bounded by the Admin ceiling
- The merge is recorded as an event in the execution graph
- Admin can reverse a merge by creating a new entity definition that splits the identities

---

## Permission System

### Discord-Style Permission Hierarchy

```
admin (root, irrevocable)
    ↓ delegates
admin_delegate (can manage, cannot supersede admin)
    ↓ delegates
trusted_user (expanded permissions, may have limited delegation)
    ↓ grants
sub_user (specific permissions, no delegation)
```

### Unified Permission Schema

```
Permissions
├── domain_access:
│   ├── technical: read | write | none
│   ├── project: read | write | none
│   ├── research: read | write | none
│   ├── interaction: read | write | none
│   ├── relationship: read | write | none
│   ├── personal: read | write | none
│   ├── private: read | write | none
│   └── credentials: read | write | none
├── privacy_ceiling: public | restricted | private | confidential
├── tool_permissions: { tool_name: allowed | denied, ... }
├── sensor_permissions: { sensor_action: allowed | denied, ... }
├── control_channel: allowed | denied
├── administrative_permissions:
│   ├── approve_channels: allowed | denied
│   ├── elevate_entities: allowed | denied
│   ├── merge_identities: allowed | denied
│   ├── manage_permissions: allowed | denied
│   └── configure_safety: allowed | denied
└── delegation:
    ├── can_delegate_to: [entity_id, ...]
    └── max_delegation_level: admin_delegate | trusted_user | sub_user
```

### Permission Enforcement

Permissions are enforced by the kernel, not by entities. Every access request passes through the permission engine:

```
Access Request: entity_id + resource_type + resource_id + action
    ↓
Permission Engine:
    Load entity permissions
    Check: entity has permission for resource + action?
    Check: permission <= Admin ceiling? (always true by construction)
    ↓
Allow or Deny + log event
```

---

## Immutability and Invalidation

- **Memories and events:** Immutable, never deleted or modified. They are the shared substrate of the runtime.
- **Knowledge:** Invalidated when source entity permissions change. If an entity is demoted, knowledge derived from that entity enters accelerated confidence decay or is restricted based on the new permission profile.
- **Macros:** Invalidated when source entity permissions change. If an entity is demoted, macros derived from that entity's execution are demoted until re-validated under the new permission profile.

This preserves the integrity of the immutable history while ensuring that derived artifacts respect current trust boundaries.

---

## Entity/Session Composition

The complete entity context at runtime is composed from three sources:

```
Entity Definition (Entity Graph, artifact store)
    +
Entity State (projection from event stream)
    +
Channel Binding (entity_bindings in Channel definition)
    =
Complete Entity/Session Context (runtime)
```

### Entity Definition

Loaded from the Entity Graph by content hash. Provides identity, trust level, permissions, and relationships. This is the "who" — the stable, versioned definition of the entity.

### Entity State

Projected from the world-state event stream by filtering all events where the entity was a subject or participant. Provides last seen, interaction count, derived preferences, and behavioral patterns. This is the "current condition" — always up-to-date but computable from history.

### Channel Binding

Loaded from the Channel definition's `entity_bindings` field. Provides the session context — which channel the entity is communicating through, what session identifier is active, and what role the entity plays in that session. This is the "how" — the communication surface and session identity.

### Composition Example

```
Entity Definition: entity_abc:v3
├── trust_level: trusted_user
├── identity: { primary: discord:@bob, merged: [voice_print_xyz] }
└── permissions: { domain_access: { technical: read, project: read }, ... }

Entity State (projected):
├── last_seen: 2024-01-15T14:30:00Z
├── interaction_count: 147
└── derived_preferences: { prefers_concise_responses: true }

Channel Binding (from channel_discord_general):
├── entity_def_hash: entity_abc:v3
├── session_id: discord:@bob
└── role: trusted_user

= Complete Entity/Session Context:
{
    entity: entity_abc:v3,
    state: { last_seen: ..., interaction_count: 147, ... },
    session: { channel: "discord_general", session_id: "discord:@bob", role: "trusted_user" },
    effective_permissions: { ... },  // entity permissions ∩ channel permissions
}
```

### Demotion and Session Composition

When an entity is demoted, the composition changes:

```
Entity Definition: entity_abc:v4 (demoted)
├── trust_level: untrusted
├── identity: { primary: discord:@bob, merged: [voice_print_xyz] }  // preserved
└── permissions: { none }

Entity State (projected):  // unchanged - all historical data persists
├── last_seen: 2024-01-15T14:30:00Z
├── interaction_count: 147
└── derived_preferences: { prefers_concise_responses: true }

Channel Binding:  // unchanged - channel still recognizes the entity
├── entity_def_hash: entity_abc:v4
├── session_id: discord:@bob
└── role: untrusted

= Complete Entity/Session Context:
{
    entity: entity_abc:v4,
    state: { last_seen: ..., interaction_count: 147, ... },  // system still knows who this is
    session: { channel: "discord_general", session_id: "discord:@bob", role: "untrusted" },
    effective_permissions: { none },  // but no runtime access
}
```

The system retains full understanding of who this entity is — identity, relationship history, accumulated knowledge — but grants no runtime access. If the entity is later re-elevated, all accumulated context is immediately available without re-derivation.

---

## Cross-References

- [TECHNICAL_CONCEPT.md](../TECHNICAL_CONCEPT.md) — Core invariants, channels, memory metadata
- [RETRIEVAL.md](RETRIEVAL.md) — How entity permissions filter memory/knowledge retrieval
- [CHANNELS.md](CHANNELS.md) — How channels bind to entities
- [SECURITY.md](SECURITY.md) — Entity trust model and threat scenarios
