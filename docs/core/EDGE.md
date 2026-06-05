# Portable Personal Edge Node (PEN)

> This document expands on the Edge Architecture section of [TECHNICAL_CONCEPT.md](../TECHNICAL_CONCEPT.md).

A **Portable Personal Edge Node** is a wearable, battery-powered distributed computing system that extends a user’s home AI infrastructure into the physical world by acting as a mobile sensor, network, and execution layer.

It consists of a **wearable compute core (bag-based node)** connected to multiple **heterogeneous sensor devices (phones, glasses, cameras, optional depth sensors, biometric inputs)** and a **user interface layer (audio, AR, and small displays)**. These components operate as a synchronized system rather than independent devices.

---

### Core Architecture

**1. Edge Perception Layer (real-time)**

- Multi-camera egocentric capture
- Depth sensing and IMU tracking
- Audio capture and translation
- Local object detection and scene parsing
- Produces structured, compressed world-state events

**2. Local Deterministic Compute Layer**

- Low-latency inference models
- Sensor fusion and SLAM
- Networking, routing, and policy enforcement
- AR rendering and interaction logic
- Ensures system remains stable even offline

**3. Remote Cognitive Layer (Homelab)**

- High-capacity reasoning and planning
- Long-term memory and trip reconstruction
- Scene re-synthesis (3D splats / neural reconstruction)
- Streaming production and content orchestration
- Identity, personality, and dialogue continuity

**4. Network Orchestration Layer**

- Multi-WAN aggregation (LTE, Wi-Fi, Ethernet)
- VPN tunnel to home as primary trusted egress
- Policy-based routing (latency vs trust vs cost)
- Always-on heartbeat maintaining global state continuity

**5. Memory & World Model**

- Continuous “heartbeat” telemetry stream
- Event-based logging (not raw media storage)
- Spatial reconstruction of visited environments
- Semantic indexing of experiences over time
- Optional 3D re-renderable “experience bubbles”

---

### Interaction Model

- **Local interface**: immediate AR/audio assistant for navigation, translation, and contextual awareness
- **Remote interface (“home assistant”)**: higher-level reasoning accessed as a call-like presence with richer dialogue and visualization
- **Machine-to-machine layer**: structured event messages between local node and homelab for coordination and memory updates

---

### Key Design Principles

- **Deterministic-first architecture** (infrastructure never depends on AI reasoning)
- **AI as interpretation layer**, not control system
- **Event-driven world-state representation** instead of raw media storage
- **Edge–cloud separation of cognition** (real-time vs reflective intelligence)
- **Progressive compression of reality** into spatial-semantic memory structures
- **Graceful degradation when disconnected**

---

### Primary Outcome

The system functions as a **personal mobile operating environment for perception and cognition**, enabling:

- augmented navigation and awareness
- live contextual translation and information overlay
- distributed multi-camera IRL streaming
- reconstructable spatial memory of lived environments
- continuous synchronization with a home-based intelligence core

---

In essence:

> A Portable Personal Edge Node is a wearable distributed computing system that turns lived experience into a continuously synchronized, semantically structured, and partially reconstructable spatial memory layer, anchored by a home AI core and executed through real-time local edge intelligence.
