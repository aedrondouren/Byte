# Entity Model

> Expands on the Entity Model section of [TECHNICAL_CONCEPT.md](../TECHNICAL_CONCEPT.md).
> **Audience:** Technical
> **Prerequisites:** TECHNICAL_CONCEPT.md Sections 1-5, 8
> **Status:** Complete

## One-Line Summary

> **Entities are lightweight graph nodes with identity, relationships, and permissions — not memory containers. All memory lives in the shared graph with metadata describing context. Entities define retrieval policies over that shared graph.**

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

```
Entity
├── id: content-addressed hash
├── type: internal | external
├── internal_type: technical_assistant | companion | stream_host | automation_agent  // internal only
├── trust_level: admin | admin_delegate | trusted_user | sub_user | untrusted
├── identity:
│   ├── primary: { channel_id, identifier }
│   └── merged: [ { channel_id, identifier, merged_at, merged_by } ]
├── relationships: [ { target_entity_id, relation_type, strength, since } ]
├── state: { last_seen, interaction_count, ... }  // projection of history, not stored separately
└── permissions: { ... }
```

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
New Participant Detected (event from channel with unknown identifier)
    ↓
Create Untrusted Entity (no permissions, no access)
    ↓
Admin or admin_delegate reviews entity
    ↓
Elevate to sub_user -> grant specific permissions
    ↓
Further elevation to trusted_user -> expanded permissions
    ↓
Possible elevation to admin_delegate -> administrative permissions (never >= Admin)
```

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
│   { channel: "discord", id: "@admin_user", merged_at: "...", merged_by: "admin" },
│   { channel: "voice", id: "voice_print_abc", merged_at: "...", merged_by: "admin" },
]
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

## Cross-References

- [TECHNICAL_CONCEPT.md](../TECHNICAL_CONCEPT.md) — Core invariants, channels, memory metadata
- [RETRIEVAL.md](RETRIEVAL.md) — How entity permissions filter memory/knowledge retrieval
- [CHANNELS.md](CHANNELS.md) — How channels bind to entities
- [SECURITY.md](SECURITY.md) — Entity trust model and threat scenarios
