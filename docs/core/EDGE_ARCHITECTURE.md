# Edge Architecture

> Expands on the Edge Architecture section of [TECHNICAL_CONCEPT.md](../TECHNICAL_CONCEPT.md#12-edge-architecture).
> **Audience:** Technical
> **Prerequisites:** TECHNICAL_CONCEPT.md Sections 1–5, GRAPH.md
> **Status:** Complete

The edge architecture is an experimental extension of the core runtime. It explores how far the compression thesis can be pushed when perception becomes mobile and continuous. The runtime's core thesis does not depend on the PEN existing.

A **Portable Personal Edge Node (PEN)** is a wearable, battery-powered distributed computing system that extends a user's home AI infrastructure into the physical world by acting as a mobile sensor, network, and execution layer.

It consists of a **wearable compute core (bag-based node)** connected to multiple **heterogeneous sensor devices** (phones, glasses, cameras, optional depth sensors, biometric inputs) and a **user interface layer** (audio, AR, and small displays). These components operate as a synchronized system rather than independent devices.

The system does not observe the world. It continuously extends its world-state graph through mobile perception nodes.

## Key Design Principles

- **Deterministic-first architecture** — infrastructure never depends on AI reasoning
- **AI as interpretation layer**, not control system
- **Event-driven world-state representation** instead of raw media storage
- **Edge-cloud separation of cognition** — real-time perception on edge, reflective intelligence on homelab
- **Progressive summarization** — raw perception becomes structured events, then narrative memories, then validated knowledge
- **Graceful degradation when disconnected** — local compute handles bounded tasks without homelab coordination

## Hardware Topology

The PEN architecture has three layers connected via VPN tunnel:

- **Homelab (Trusted Core):** Cognitive Core, Long-term Memory, Code Registry
- **Network:** Multi-WAN aggregation across LTE, WiFi, and Ethernet. VPN tunnel to homelab is the primary trusted egress.
- **Portable Personal Edge Node (PEN):**
  - Compute Core (backpack/bag): Perception Processing, Local Inference, Network Orchestration
  - Sensor Devices: Glasses (camera), Phone (compute), Biometric sensors (EEG/EMG)

## Edge Perception Layer

Real-time multi-camera egocentric capture, depth sensing and IMU tracking, audio capture and translation, local object detection and scene parsing. Produces structured perception — the first entries in the world-state graph. Raw sensor streams never leave the edge; only structured perception enters the graph.

## Local Deterministic Compute Layer

Low-latency inference models, sensor fusion and SLAM, networking and routing, policy enforcement, AR rendering and interaction logic. Ensures the system remains stable even when offline.

### Power and Thermal Constraints

The PEN operates on battery power with constrained thermal envelope. This imposes specific design requirements:

- **Compute allocation is power-aware.** The scheduler adjusts inference capacity based on battery level and thermal headroom.
- **Local models are optimized for efficiency.** Smaller, quantized models run on edge; heavy models are offloaded to the homelab.
- **Graceful degradation includes power states.** At low battery, the PEN disables non-essential perception modalities and maintains only safety-critical functions.

### Network Bandwidth Requirements

| Modality             | Local Processing                 | Homelab Transmission                     |
| -------------------- | -------------------------------- | ---------------------------------------- |
| Camera (RGB + depth) | Object detection, scene parsing  | Structured perception events (~1-10/sec) |
| Audio                | Speech-to-text                   | Transcription events (~1/sec)            |
| IMU                  | Movement pattern detection       | Locomotion events (~1/sec)               |
| EEG/EMG              | Attention, fatigue, confirmation | Derived state indices (~0.1/sec)         |
| Gaze                 | Fixation detection               | Attention events (~1-5/sec)              |

Raw sensor data never crosses the network boundary. Only structured events are transmitted, keeping bandwidth requirements low (~KB/sec, not MB/sec).

## Remote Cognitive Layer (Homelab)

High-capacity reasoning and planning, long-term memory and trip reconstruction, scene re-synthesis through 3D splats or neural reconstruction, streaming production and content orchestration, identity, personality, and dialogue continuity.

## Network Orchestration Layer

Multi-WAN aggregation across LTE, Wi-Fi, and Ethernet. VPN tunnel to home as the primary trusted egress. Policy-based routing balancing latency versus trust versus cost. Always-on heartbeat maintaining global state continuity.

## Memory and World Model

Continuous heartbeat telemetry stream. Event-based logging rather than raw media storage. Spatial reconstruction of visited environments. Semantic indexing of experiences over time. Optional 3D re-renderable experience bubbles.

## Interaction Model

**Local interface** — runs on edge compute for low-latency response. The user interacts with the PEN directly without homelab dependency. Immediate AR/audio assistant for navigation, translation, and contextual awareness.

**Remote interface ("home assistant")** — routes to the homelab cognitive core. The user "calls home" when they need deeper reasoning or access to long-term memory. Higher-level reasoning accessed as a call-like presence with richer dialogue and visualization.

**Machine-to-machine layer** — the homelab sends execution chains, context updates, and knowledge queries; the PEN sends structured perception and execution outcomes. No raw media crosses this boundary.

## Graceful Degradation

When the PEN disconnects from the homelab, it continues to function with local compute only:

- **What works offline:** perception processing, local inference, immediate interaction logic, safety-critical chains, bounded automation tasks.
- **What does not work offline:** heavy reasoning, long-term memory access, planning, code registry access, macro discovery, knowledge validation.
- **Safety guarantees:** critical chains remain non-interruptible. Actions requiring homelab verification are deferred, never executed speculatively.

## Local Model Specifications

| Model Type                    | Runs On         | Purpose                                          |
| ----------------------------- | --------------- | ------------------------------------------------ |
| Object detection (YOLO-style) | Edge            | Real-time scene parsing                          |
| Speech-to-text                | Edge            | Audio transcription                              |
| Attention tracking            | Edge            | Gaze fixation detection                          |
| Small reasoning model         | Edge (optional) | Bounded local reasoning when homelab unavailable |
| Heavy reasoning model         | Homelab         | Planning, summarization, knowledge validation    |

## Essence Statement

> A Portable Personal Edge Node is a wearable distributed computing system that turns lived experience into a continuously synchronized, semantically structured, and partially reconstructable spatial memory layer, anchored by a home AI core and executed through real-time local edge intelligence.

---

**Related:** [TECHNICAL_CONCEPT.md](../TECHNICAL_CONCEPT.md#12-edge-architecture) for the original specification.
