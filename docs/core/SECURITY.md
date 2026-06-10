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

Macros are compiled from execution traces and represent reusable behavior patterns. Macro definitions are stored in the artifact version store; execution traces are recorded in the world-state event store. A malicious or corrupted macro could encode harmful behavior. All macros must pass validation tests before activation. Macros must be fully reversible — the system must be able to decompose any macro back into its constituent primitives and verify its behavior. Macro execution must be auditable through the trace IR, and any macro can be demoted or disabled at any time.

### Skill-Derived Macro Validation

Macros derived from skill execution traces carry provenance links to their source skill. Validation includes an additional check: the macro's behavior must be equivalent to the source skill's intent, not just the observed trace. This prevents overfitting to a specific execution instance. When a skill is updated, all derived macros are flagged for re-validation against the new version. The provenance chain ensures that no derived artifact executes with stale source references. Macro definitions (artifact store) reference execution traces (event store) for provenance.

## Skill Registry Integrity

Skills are pre-authored behavior definitions installed by the user from any source, stored in the artifact version store. The system does not curate or approve skills. Trust is the user's responsibility. The following integrity measures apply:

- **Content-addressed storage.** Every skill version is identified by its cryptographic hash (SHA-256). Once published, a skill version cannot be changed. The hash is the canonical identifier; semantic version tags are mutable pointers.
- **Provenance tracking.** All macros and knowledge artifacts derived from skill execution carry immutable provenance metadata linking them to the specific skill version that produced them. Cross-store references link artifact versions to world-state events.
- **Version isolation.** When a skill is updated, the new version is stored as a separate artifact. Old versions remain available for derived artifact re-validation and audit.
- **Open ecosystem boundary.** Skills from untrusted sources execute within the same RPU contract as all other reasoning — the kernel validates outputs, tool calls are authorized, and state updates are verified. A malicious skill cannot execute arbitrary actions; it can only propose intent through the structured RPU contract.

## Skill Security Audit

The offline optimization loop includes a Skill Security Audit activity that analyzes installed skills for known risk patterns. This is advisory — it raises events but does not block skill execution. The audit operates on the skill's instruction file and tool reference declarations, not on runtime behavior.

**Audit categories:**

- **Prompt injection vectors** — skill instructions that attempt to override system prompts, escape the RPU contract, or manipulate the kernel's validation logic.
- **Dangerous tool access** — skills that request access to tools with destructive capabilities (file deletion, system modification, external API calls with write access). Flagged for user review.
- **Data exfiltration patterns** — skills that attempt to send structured data to external endpoints not declared in the skill's tool references.
- **Privilege escalation attempts** — skills that attempt to reference kernel-level functions, bypass the scheduler, or access internal runtime state.

**Audit findings** are recorded as events in the execution graph with the following structure:

```json
{
	"type": "security_audit_finding",
	"skill_id": "skill_example",
	"skill_version": "1.0.0",
	"category": "prompt_injection | dangerous_tool_access | data_exfiltration | privilege_escalation",
	"severity": "low | medium | high | critical",
	"description": "Skill instruction contains pattern matching known injection technique...",
	"detected_at": "2024-03-15T03:00:00Z"
}
```

**User notification.** Audit findings are surfaced through the active channel. The user decides whether to uninstall the skill, modify it, or ignore the finding. The system does not take automatic action on audit findings.

**Limitations.** The audit is heuristic-based. False positives (flagging benign skills) and false negatives (missing malicious skills) are both possible. The audit complements but does not replace the user's judgment. The RPU contract and kernel validation provide the primary defense — even a malicious skill cannot execute arbitrary actions.

## Code Registry Integrity

Code components in the registry are stored in the artifact version store — versioned, content-addressed, and cryptographically verifiable. Every component version is immutable once published. The test pipeline ensures that only validated code enters the registry. Cross-store references link component versions to their test runs in the world-state event store, providing full auditability. Edge nodes verify component integrity through cryptographic hashes before making components available for code generation.

## Entity Trust Model

The system uses a hierarchical trust model with Admin as the single root of trust:

- **Admin** exists from bootstrap, pre-linked to the primary webUI. Irrevocable, cannot be superseded.
- **Admin Delegate** can manage the system but never supersede Admin.
- **Trusted User** has expanded permissions with possible limited delegation.
- **Sub-User** has limited, specific permissions granted by a higher-trust entity.
- **Untrusted** is the default for all new entities — no access to any resource.

Trust is never automatic. No entity becomes trusted through behavior alone. Trust is explicitly granted by a higher-trust entity. The Admin ceiling invariant ensures no delegation chain can produce permissions equal to or greater than Admin.

## Permission Enforcement

All access — domains, tools, sensors, control signals — is gated by entity permissions enforced by the kernel:

- **Domain access** controls which memory/knowledge domains an entity can read or write.
- **Privacy ceiling** limits the maximum privacy level an entity can access.
- **Tool permissions** gate access to individual tool capabilities.
- **Sensor permissions** distinguish between read (receiving perception events) and control (sending sensor commands).
- **Control channel authority** must be explicitly enabled per channel (default: disabled).

The permission engine validates every access request. Denied access is logged as an event for audit.

## Channel Security

Channels are trust boundaries between the runtime and external surfaces. No channel processes events until approved by Admin or admin_delegate with `approve_channels` permission. Unapproved channels exist in `pending_approval` status with no event processing capability.

Channels are not control channels by default — even Admin-connected channels must be explicitly enabled to send kernel control signals. This provides a safety layer: if an Admin session is compromised on a secondary channel, the attacker cannot send kernel control signals.

Input from any channel is treated as an intent proposal, not a command — the kernel validates and schedules execution regardless of source. Channels on untrusted surfaces (public APIs, third-party messaging platforms) must never receive raw world-state data, only projected and filtered views appropriate to that channel's entity permissions.

## Identity Merge Security

Identity merging is a security-sensitive operation because it grants one entity access to another entity's memories and knowledge:

- Admin can merge any identities.
- Admin delegates can merge identities within their permission scope.
- Trusted users with `merge_identities` permission can only merge their own identities (self-merge across channels).
- Sub-users cannot merge identities.

All merges are logged as events. Admin is notified of all merges, even delegated ones. Admin can reverse merges.

## Contextual Metadata Protection

The dual-access knowledge model prevents accidental leakage of contextual information:

- Factual knowledge content propagates globally when domain permissions allow.
- Contextual metadata (who taught this, under what relationship) remains scoped to the originating relationship.
- The retrieval pipeline strips or anonymizes contextual metadata when knowledge is accessed outside its originating relationship.

## Observability and Audit

All execution is trace-driven. The trace IR provides a complete audit log of every action taken by the system. This audit trail is tamper-evident through cryptographic hashing and accessible for review. Users must be able to inspect the full execution history, understand why actions were taken, and trace any behavior back to its originating intent proposal and sensor inputs.

All permission changes, channel approvals, entity elevations, and identity merges are recorded as events in the execution graph with full provenance.

---

**Related:** [THREAT_MODEL.md](THREAT_MODEL.md#assumed-attacker-capabilities) for attacker capability analysis and threat scenarios. [TECHNICAL_CONCEPT.md](../TECHNICAL_CONCEPT.md#15-security-and-privacy-considerations) for the original specification.
