# Channel Architecture

> Expands on the Channel Architecture section of [TECHNICAL_CONCEPT.md](../TECHNICAL_CONCEPT.md).
> **Audience:** Technical
> **Prerequisites:** TECHNICAL_CONCEPT.md Sections 1-5, 8; [ENTITIES.md](ENTITIES.md)
> **Status:** Complete

## One-Line Summary

> **All channels are bidirectional event surfaces that must be Admin-approved before activation. Channels produce events into the system and consume projected views from it. Control signal authority requires explicit enablement (default: disabled). The event surface of a channel is analogous to a tool definition — it declares what it produces and what it consumes.**

---

## Core Principle

Channels are bidirectional boundaries between the runtime and external surfaces. Every channel both produces events into the system and consumes projected views from it. The primary orientation (input vs. output) determines the dominant flow, but both directions are always available.

```
Every Channel:
    -> Kernel: Events (perception, user messages, control signals)
    <- Kernel: Events (world-state projections, sensor controls)

Primary orientation determines dominant flow:
    Input-oriented: perception events IN, sensor control OUT
    Output-oriented: world-state projections OUT, user events IN
```

Channels do not store state. They translate between external surfaces and the internal event model. Logically, both sides see events — channels may aggregate events to present a "current state" view to users, but the underlying model is always event-based.

---

## Channel Types

### Input-Oriented Channels

Primary flow: perception events into the system.

Examples: cameras, microphones, voice interfaces, sensor arrays.

```
Input Channel: camera_front
    -> Kernel: perception_event (object_detection, scene_analysis)
    <- Kernel: sensor_control (pan, tilt, zoom, resolution)
```

### Output-Oriented Channels

Primary flow: world-state projections out to external surfaces.

Examples: Discord bots, web UIs, API endpoints, notification services.

```
Output Channel: discord_general
    -> Kernel: user_message, channel_reaction
    <- Kernel: world_state_projection (text_response, status_update)
```

### Bidirectional Channels

All channels are technically bidirectional. The distinction between input and output is about primary orientation, not capability. A webUI, for example, is heavily bidirectional — it displays world-state projections and accepts user messages and admin control signals.

---

## Channel Approval Workflow

No channel processes events until approved by Admin or an admin_delegate with `approve_channels` permission.

```
New Channel Connects
    ↓
Reports capabilities:
    ├── produces: [event_type, ...]
    └── consumes: [event_type, ...]
    ↓
Status: pending_approval
    ├── No event processing
    ├── No graph access
    ├── Admin notification triggered
    └── Events logged but not processed
    ↓
Admin or admin_delegate reviews via webUI:
    ├── Channel type and capabilities
    ├── Entity bindings (fixed) or resolution method (dynamic)
    ├── Domain access and privacy ceiling
    ├── Control channel flag (default: false)
    └── Sensor control scope (if applicable)
    ↓
Admin approves -> Status: active
    ↓
Bidirectional event flow begins, filtered by entity permissions
```

### Channel Schema

```
Channel
├── id: content-addressed hash
├── surface: "discord" | "webui" | "camera" | "microphone" | "voice" | "api" | ...
├── status: pending_approval | active | suspended | revoked
├── approved_by: entity_id
├── approved_at: timestamp
├── entity_bindings: [ { entity_id, session_id, role } ]        // fixed channels
├── entity_resolution: { method, fallback: untrusted_entity }   // dynamic channels
├── produces: [event_type, ...]
├── consumes: [event_type, ...]
├── permissions:
│   ├── allowed_domains: [domain, ...]
│   ├── blocked_domains: [domain, ...]
│   ├── max_privacy_level: privacy_level
│   ├── control_channel: true | false
│   ├── control_signals: [signal_type, ...]
│   └── sensor_controls: [sensor_id, ...]
└── metadata: { capabilities, trust_vector, ... }
```

---

## Entity Binding

### Fixed Channels

Fixed channels are bound to specific entities and sessions at configuration time.

Example: a Discord DM channel bound to a specific user.

```
Channel: discord_dm_admin
├── surface: "discord"
├── entity_bindings: [
│   { entity_id: "entity_abc", session_id: "discord:@admin_user", role: "admin" }
│ ]
├── produces: [user_message, channel_reaction]
└── consumes: [world_state_projection]
```

### Dynamic Channels

Dynamic channels resolve entity identity at event time, using voice prints, face recognition, manual ID tokens, or contextual inference.

Example: a speech-to-text channel that identifies speakers dynamically.

```
Channel: voice_stt_living_room
├── surface: "voice"
├── entity_resolution: {
│   method: "voice_print",
│   fallback: "untrusted_entity"
│ }
├── produces: [voice_transcript, speaker_identified]
└── consumes: [world_state_projection]
```

When identification fails, the event is attributed to the fallback entity (typically untrusted).

---

## Control Channel Flag

Channels are **not control channels by default**, even for Admin-connected channels. A channel must be explicitly enabled to send kernel control signals.

```
Channel: webui_admin_primary
├── entity: admin
├── control_channel: true        // explicitly enabled
├── control_signals: [all]
└── ...

Channel: webui_admin_secondary
├── entity: admin
├── control_channel: false       // Admin identity but no control through this channel
└── ...
```

This provides a safety layer: even if an Admin session is compromised on a secondary channel, the attacker cannot send kernel control signals.

### Control Signal Taxonomy

#### Kernel Control Signals (channel -> kernel, requires control_channel: true)

| Signal             | Purpose                    | Effect                     |
| ------------------ | -------------------------- | -------------------------- |
| `pause_chain`      | Suspend execution chain    | Chain -> parked state      |
| `resume_chain`     | Resume parked chain        | Chain -> active state      |
| `terminate_chain`  | End execution chain        | Chain -> terminated        |
| `escalate_chain`   | Elevate chain priority     | Priority tier increased    |
| `switch_context`   | Change active task context | ContextProjection updated  |
| `request_status`   | Request execution recap    | Generates status response  |
| `approve_proposal` | Approve a pending proposal | Proposal -> commit         |
| `reject_proposal`  | Reject a pending proposal  | Proposal -> discarded      |
| `elevate_entity`   | Elevate entity trust level | Entity permissions updated |
| `approve_channel`  | Approve pending channel    | Channel -> active          |
| `merge_identities` | Merge entity identities    | Identity graph updated     |
| `demote_entity`    | Revoke entity trust        | Entity permissions reduced |

#### Sensor Control Signals (kernel -> input channel)

| Signal              | Purpose                        |
| ------------------- | ------------------------------ |
| `sensor_pan`        | Adjust camera horizontal angle |
| `sensor_tilt`       | Adjust camera vertical angle   |
| `sensor_zoom`       | Adjust camera focal length     |
| `sensor_resolution` | Change capture quality         |
| `sensor_mute`       | Disable audio capture          |
| `sensor_activate`   | Enable dormant sensor          |
| `sensor_deactivate` | Disable active sensor          |

#### Regular Events (any channel -> kernel, no control_channel required)

| Event Type         | Description                        |
| ------------------ | ---------------------------------- |
| `user_message`     | Text message from user             |
| `voice_transcript` | Speech-to-text output              |
| `gesture_event`    | Detected gesture                   |
| `channel_reaction` | Emoji reaction, etc.               |
| `perception_event` | Structured perception from sensors |

---

## Channel Event Surface as Tool Definition

Channels declare their capabilities in the same way tools declare theirs. The event surface of a channel is analogous to a tool definition:

```
Channel: camera_front
├── produces: [perception_event: object_detection, scene_analysis]
├── consumes: [sensor_control: pan, tilt, zoom, resolution]
└── safety_constraints: { Admin-configured }

Channel: discord_general
├── produces: [user_message, channel_reaction]
├── consumes: [world_state_projection: text_response]
└── safety_constraints: { max_privacy_level: public }
```

This unifies the model: channels, tools, and sensors are all capabilities with permissions.

---

## Channel View Aggregation

Logically, both sides of a channel see events. However, channels may aggregate events to present a "current state" view to users. This is a channel-level concern, not a kernel concern.

```
Kernel -> Channel: [event_1, event_2, event_3, ...]
Channel aggregates -> User sees: "Current state: X"
```

The kernel does not manage how channels present state to users. It only sends events. Channels are responsible for their own caching, memoization, and state aggregation.

---

## Channel Lifecycle States

| State              | Description                                                             |
| ------------------ | ----------------------------------------------------------------------- |
| `pending_approval` | Channel connected, awaiting Admin review. No event processing.          |
| `active`           | Channel approved, bidirectional event flow active.                      |
| `suspended`        | Channel temporarily disabled by Admin. Events logged but not processed. |
| `revoked`          | Channel permanently disabled. Connection terminated.                    |

Channel state changes are recorded as events in the execution graph.

---

## Cross-References

- [TECHNICAL_CONCEPT.md](../TECHNICAL_CONCEPT.md) — Original channel specification, projections vs. channels
- [ENTITIES.md](ENTITIES.md) — Entity model, trust levels, permission system
- [RETRIEVAL.md](RETRIEVAL.md) — How channel policies filter retrieval
- [SECURITY.md](SECURITY.md) — Channel security and threat scenarios
