# Security and Privacy

> Expands on the Security and Privacy section of [TECHNICAL_CONCEPT.md](../TECHNICAL_CONCEPT.md#15-security-and-privacy-considerations).
> **Audience:** Technical
> **Prerequisites:** TECHNICAL_CONCEPT.md Sections 1–5, GRAPH.md
> **Status:** Complete

## Data Boundaries

The system spans edge devices, home infrastructure, and potentially cloud providers. Data boundaries must be clearly enforced. Raw sensor data, especially biometric data, is processed at the edge whenever possible. Only structured, compressed world-state events are transmitted to the homelab. Cloud providers receive only normalized inference requests through the LLM Runtime, with no access to raw world-state or execution traces.

## Biometric Data Handling

EEG, EMG, gaze tracking, and other biometric inputs are highly sensitive personal data. These signals must never leave the edge node in raw form. They are processed locally into derived state estimates — attention level, fatigue index, confirmation signals — which are then fused into the world-state graph. Raw biometric streams are never stored, transmitted, or logged. Derived estimates are ephemeral and tied to the current session context.

## Network Security

The PEN maintains a VPN tunnel to the homelab as its primary trusted egress. Multi-WAN aggregation across LTE, Wi-Fi, and Ethernet introduces multiple attack surfaces. Policy-based routing must enforce that all traffic to the homelab traverses the encrypted tunnel. Fallback routing through untrusted networks must be restricted to essential heartbeat signals only, with no world-state or sensor data exposed.

## Trust Boundaries Between Nodes

Each node in the distributed system operates with a different trust level. The homelab is the trusted core. The PEN is a semi-trusted mobile node subject to physical compromise. Cloud inference providers are untrusted third parties. The system must enforce mutual authentication between all nodes, encrypt all inter-node communication, and ensure that no single node compromise exposes the full world-state or execution history.

## Cryptographic Integrity

Events in the world-state graph are cryptographically hashed and optionally signed. This creates a tamper-evident history where any modification to past events is detectable. The hash chain ensures causal lineage — each event references its parents, making the full provenance chain verifiable.

## Graceful Degradation and Offline Safety

When the PEN disconnects from the homelab, it must continue to function safely with local compute only. This means local inference models must have bounded capabilities that do not require homelab coordination. Critical safety chains must remain non-interruptible even in degraded mode. Any action that requires homelab verification must be deferred or denied when offline, never executed speculatively.

## Macro Validation and Reversibility

Macros are compiled from execution traces and represent reusable behavior patterns. A malicious or corrupted macro could encode harmful behavior. All macros must pass validation tests before activation. Macros must be fully reversible — the system must be able to decompose any macro back into its constituent primitives and verify its behavior. Macro execution must be auditable through the trace IR, and any macro can be demoted or disabled at any time.

## Code Registry Integrity

Code components in the registry are versioned, content-addressed, and cryptographically verifiable. Every component version is immutable once published. The test pipeline ensures that only validated code enters the registry. Cross-graph references link component versions to their test runs in the world-state graph, providing full auditability. Edge nodes verify component integrity through cryptographic hashes before making components available for code generation.

## Channel Security

Channels are trust boundaries between the runtime and external surfaces. Each channel enforces its own authentication, authorization, and rate limiting. Input from any channel is treated as an intent proposal, not a command — the kernel validates and schedules execution regardless of source. Channels on untrusted surfaces (public APIs, third-party messaging platforms) must never receive raw world-state data, only projected and filtered views appropriate to that channel's trust level.

## Observability and Audit

All execution is trace-driven. The trace IR provides a complete audit log of every action taken by the system. This audit trail is tamper-evident through cryptographic hashing and accessible for review. Users must be able to inspect the full execution history, understand why actions were taken, and trace any behavior back to its originating intent proposal and sensor inputs.

---

**Related:** [THREAT_MODEL.md](THREAT_MODEL.md#assumed-attacker-capabilities) for attacker capability analysis and threat scenarios. [TECHNICAL_CONCEPT.md](../TECHNICAL_CONCEPT.md#15-security-and-privacy-considerations) for the original specification.
