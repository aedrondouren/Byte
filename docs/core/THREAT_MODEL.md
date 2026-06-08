# Threat Model

> Attacker capability analysis and threat scenarios for the B.Y.T.E. architecture.
> **Audience:** Technical / Security
> **Prerequisites:** SECURITY.md, TECHNICAL_CONCEPT.md Sections 1–5, 15
> **Status:** Complete

## Assumed Attacker Capabilities

| Attacker Type                 | Capabilities                                                                       | Trust Level                |
| ----------------------------- | ---------------------------------------------------------------------------------- | -------------------------- |
| **External network attacker** | Network interception, MITM, DDoS, port scanning                                    | Untrusted                  |
| **Physical attacker**         | Physical access to edge device, hardware tampering                                 | Untrusted                  |
| **Compromised edge node**     | Full control of PEN, access to local storage and compute                           | Semi-trusted (compromised) |
| **Compromised LLM provider**  | Manipulated model outputs, prompt injection, training data extraction              | Untrusted                  |
| **Malicious insider**         | Access to homelab, ability to modify code registry, artifact store, or event store | Trusted (but monitored)    |
| **Supply chain attacker**     | Compromised dependencies, poisoned code components                                 | Untrusted                  |

## Threat Scenarios

### T1: Network Interception of Edge-Homelab Communication

**Attack:** Intercept and modify structured events flowing from the PEN to the homelab.

**Impact:** Modified perception events could trigger incorrect situation models, intents, and executions. An attacker could inject false perceptions or suppress real ones.

**Mitigations:**

- VPN tunnel for all edge-homelab communication (SECURITY.md Section 15.3).
- Mutual authentication between nodes (SECURITY.md Section 15.4).
- Event cryptographic hashes detect modification. Modified events fail hash verification.
- Heartbeat monitoring detects communication anomalies.

**Residual risk:** Low. VPN + mutual authentication + hash verification provides defense in depth.

### T2: Physical Tampering with Edge Device

**Attack:** Gain physical access to the PEN and extract stored data or modify local behavior.

**Impact:** Access to stored events, memories, and knowledge. Potential modification of local perception processing or inference models.

**Mitigations:**

- Raw biometric data is never stored (SECURITY.md Section 15.2).
- Local storage is encrypted at rest.
- Tamper-evident event hashes detect modification of stored events.
- Graceful degradation limits damage from compromised local models (SECURITY.md Section 15.6).
- The PEN is semi-trusted; the homelab validates all incoming events.

**Residual risk:** Medium. Physical access is a strong attack vector. Encryption and tamper-evidence reduce but do not eliminate risk.

### T3: LLM Provider Compromise / Jailbreak

**Attack:** The LLM provider's model is compromised, jailbroken, or manipulated to produce malicious outputs.

**Impact:** Malicious RPU responses could propose harmful actions, exfiltrate information through outputs, or corrupt world-state through invalid state updates.

**Mitigations:**

- The kernel validates all RPU outputs against expected schemas before use (MULTIMODAL_INTERFACE.md: Orchestration Harness).
- Tool calls proposed by the RPU are verified against current world-state and priority constraints before execution.
- State updates proposed by the RPU are validated against the current world-state projection before being committed.
- The RPU cannot execute actions directly; it can only propose. The kernel is the execution authority.
- Model versioning enables rollback to a known-good model version (RPU.md: Model Versioning Strategy).

**Residual risk:** Low. The "AI proposes; kernel executes" invariant (TECHNICAL_CONCEPT.md Section 1.4) is the primary defense. Even a fully compromised model cannot execute arbitrary actions.

### T4: Prompt Injection Through Channel Input

**Attack:** A user or external system sends input through a channel that contains prompt injection, attempting to manipulate the RPU's behavior.

**Impact:** The RPU could be tricked into producing incorrect outputs, leaking context information, or proposing harmful actions.

**Mitigations:**

- All channel input is treated as intent proposals, not commands. The kernel validates and schedules execution regardless of source (SECURITY.md Section 15.9).
- Input validation ensures all data passed to the RPU is structured and schema-validated (MULTIMODAL_INTERFACE.md: Orchestration Harness).
- Channels on untrusted surfaces never receive raw world-state data, only projected and filtered views.
- The RPU receives structured context, not raw user input. User input is processed through the signal-to-intent pipeline before reaching the RPU.

**Residual risk:** Low. Structured input + kernel validation + channel isolation provides defense in depth.

### T5: Malicious Code Component in Registry

**Attack:** A malicious or compromised code component is published to the registry and used in generated software.

**Impact:** Generated software could contain backdoors, data exfiltration, or other malicious behavior.

**Mitigations:**

- Code components are stored in the artifact version store with mandatory test pipeline validation before adoption (TECHNICAL_CONCEPT.md Section 9.2.2).
- Components are content-addressed and immutable once published (SECURITY.md Section 15.8).
- Edge nodes verify component integrity through cryptographic hashes before use.
- Cross-store references link component versions to their test runs in the world-state event store, providing full auditability.
- Deprecation policy allows rapid removal of compromised components (REGISTRY.md: Component Deprecation Policy).

**Residual risk:** Low. Test pipeline + immutability + hash verification + auditability provides strong protection. However, the test pipeline's coverage depends on test quality.

### T6: Supply Chain Attack on External Dependencies

**Attack:** An external dependency (framework, library, driver) is compromised.

**Impact:** The compromised dependency could affect the runtime, perception processing, or generated software.

**Mitigations:**

- External dependencies are clearly separated from internal logic (REGISTRY.md: Dependency Taxonomy).
- The code registry provides tested, versioned internal alternatives for common patterns, reducing reliance on external packages.
- Dependency version pinning ensures reproducible builds.
- The test pipeline catches regressions from dependency updates.

**Residual risk:** Medium. External dependencies are inherently risky. Version pinning and testing reduce but do not eliminate risk.

### T7: World-State Event Store Corruption

**Attack:** The world-state event store is corrupted through hardware failure, software bug, or malicious modification.

**Impact:** Loss of provenance, incorrect state projections, potential system failure. The artifact version store remains unaffected.

**Mitigations:**

- Events are cryptographically hashed. Corruption is detectable through hash verification.
- Checkpoints accelerate reconstruction from uncorrupted history segments.
- The append-only model reduces corruption risk (no in-place modification).
- Regular backups of the event store to separate storage.
- The artifact version store has independent integrity verification and backup.

**Residual risk:** Low. Hash detection + checkpoints + backups provides strong protection. Recovery from corruption is possible through event replay.

### T8: Macro Execution Attack

**Attack:** A malicious or corrupted macro is promoted and executes harmful behavior. Macro definitions are stored in the artifact version store; execution traces are recorded in the world-state event store.

**Impact:** The macro could perform unauthorized tool calls, exfiltrate data, or corrupt world-state.

**Mitigations:**

- All macros pass validation tests before activation (SECURITY.md Section 15.7).
- Macros are fully reversible — they can be decomposed into constituent primitives and verified.
- Macro execution is auditable through the trace IR in the world-state event store.
- Any macro can be demoted or disabled at any time in the artifact version store.
- Macros use Tool Services only, which are bounded by the kernel's execution model.
- Cross-store references link macro versions to their execution traces for full provenance.

**Residual risk:** Low. Validation + reversibility + auditability + demotion provides strong protection.

### T9: Unauthorized Channel Activation

**Attack:** An unapproved channel attempts to send events or access the system.

**Impact:** Untrusted input could corrupt perception, inject malicious intent proposals, or access restricted world-state data.

**Mitigations:**

- Default-deny: channels in `pending_approval` status have no event processing capability (CHANNELS.md).
- Events from unapproved channels are logged but not processed.
- Admin notification triggered on new channel connection.
- Channel approval requires Admin or admin_delegate with `approve_channels` permission.

**Residual risk:** Low. Default-deny model with explicit approval required.

### T10: Entity Privilege Escalation

**Attack:** A sub-user or trusted user attempts to access domains, privacy levels, tools, or sensors beyond their permissions.

**Impact:** Unauthorized access to private memories, credentials, or destructive tool capabilities.

**Mitigations:**

- Retrieval pipeline enforces permissions at query time (RETRIEVAL.md).
- Permission engine validates every access request (ENTITIES.md).
- All access attempts are logged as events with full provenance.
- Admin ceiling invariant ensures no delegation chain exceeds Admin permissions.

**Residual risk:** Low. Permissions are enforced by the kernel, not by entities.

### T11: Identity Spoofing

**Attack:** An entity attempts to impersonate another entity by using their identifier.

**Impact:** Access to another entity's memories, knowledge, and permissions.

**Mitigations:**

- Channel-level authentication (Discord OAuth, webUI session tokens, voice print verification).
- Identity resolution flags anomalies (same identifier from different channels, sudden behavior changes).
- Merge events require Admin or delegated authorization.

**Residual risk:** Medium. Depends on authentication strength of each channel surface.

### T12: Malicious Identity Merge

**Attack:** A sub-user or admin_delegate with `merge_identities` permission merges their identity with a higher-privilege entity to gain access.

**Impact:** Unauthorized access to higher-privilege entity's memories, knowledge, and permissions.

**Mitigations:**

- Merge operations require Admin confirmation or at minimum Admin notification.
- Trusted users can only self-merge (own identities across channels).
- Merge events are recorded with full provenance.
- Admin can reverse merges.

**Residual risk:** Low. Admin oversight on all merges.

### T13: Control Channel Hijacking

**Attack:** An attacker gains access to a channel and attempts to send kernel control signals.

**Impact:** Unauthorized pause/terminate of execution chains, entity elevation, channel approval, or identity manipulation.

**Mitigations:**

- Control channel flag is disabled by default, even for Admin-connected channels.
- Control signals require explicit channel enablement by Admin.
- Channel-level authentication provides additional verification.
- All control signal events are logged with entity provenance.

**Residual risk:** Low. Defense in depth: control flag + authentication + audit.

### T14: Unauthorized Sensor Control

**Attack:** An entity attempts to control sensors (camera pan/tilt/zoom, mic mute) without permission.

**Impact:** Privacy violation (pointing camera at private areas), disruption (muting microphones), or surveillance.

**Mitigations:**

- Sensor permissions enforced by kernel (read vs. control separated).
- Sensor control events recorded in execution graph.
- Admin-configured safety constraints on sensor behavior.

**Residual risk:** Low. Permission-based enforcement with audit trail.

### T15: Contextual Metadata Leakage

**Attack:** Knowledge learned in a private context is accessed by an entity that should not know the provenance.

**Impact:** Leakage of who taught what, under what relationship, or in what context.

**Mitigations:**

- Dual-access knowledge projection strips contextual metadata for non-originating entities (RETRIEVAL.md).
- Factual content propagates; contextual metadata remains scoped.
- Retrieval pipeline enforces metadata stripping at projection time.

**Residual risk:** Low. Enforced by the retrieval pipeline, not by application logic.

### T16: Admin Delegate Overtake Attempt

**Attack:** An admin_delegate attempts to elevate their own permissions to equal or exceed Admin.

**Impact:** Full system compromise.

**Mitigations:**

- Admin ceiling invariant enforced by construction (ENTITIES.md).
- Admin trust level is singular and irrevocable.
- Permission delegation always produces strict subsets.
- All permission changes are logged and auditable.

**Residual risk:** Low. Enforced by system architecture, not by policy.

## Trust Boundary Summary

```
┌─────────────────────────────────────────────────────────┐
│ UNTRUSTED                                               │
│  ┌───────────────┐  ┌───────────────┐  ┌─────────────┐  │
│  │ LLM Provider  │  │ External APIs │  │ Unapproved  │  │
│  │ (compromised) │  │ (compromised) │  │ Channels    │  │
│  └───────┬───────┘  └───────┬───────┘  └──────┬──────┘  │
│     normalized          filtered        no processing   │
│     requests only       views only            │         │
├──────────┼──────────────────┼─────────────────┼─────────┤
│ PENDING  │                  │                 │         │
│  ┌───────┴──────────────────┴─────────────────┴──────┐  │
│  │ Pending Channels (awaiting Admin approval)        │  │
│  │ ↓ no event processing, no graph access            │  │
├──┼───────────────────────────────────────────────────┼──┤
│  │ SEMI-TRUSTED                                      │  │
│  │  ┌─────────────────────────────────────────────┐  │  │
│  │  │ Sub-Users (elevated external entities)      │  │  │
│  │  │ ↓ domain-restricted, privacy-capped         │  │  │
│  │  │ Edge Node (PEN)                             │  │  │
│  │  │ ↓ structured events only                    │  │  │
│  │  └─────────────────────────────────────────────┘  │  │
├──┼───────────────────────────────────────────────────┼──┤
│  │ TRUSTED                                           │  │
│  │  ┌─────────────────────────────────────────────┐  │  │
│  │  │ Admin (primary webUI + merged identities)   │  │  │
│  │  │ ↓ full access, all domains, all privacy     │  │  │
│  │  │ Homelab (Trusted Core)                      │  │  │
│  │  │ ↓ Event Store + Kernel + Memory + Knowledge │  │  │
│  │  │ All execution validated, all events hashed  │  │  │
│  │  └─────────────────────────────────────────────┘  │  │
└──┴───────────────────────────────────────────────────┴──┘

Trust flows: Admin approves channels -> channels bind to entities ->
entities have permissions -> permissions filter retrieval
```

## Security Posture Summary

| Threat                              | Likelihood | Impact   | Mitigation Effectiveness | Residual Risk |
| ----------------------------------- | ---------- | -------- | ------------------------ | ------------- |
| T1: Network interception            | Medium     | High     | High                     | Low           |
| T2: Physical tampering              | Low        | High     | Medium                   | Medium        |
| T3: LLM compromise                  | Medium     | High     | High                     | Low           |
| T4: Prompt injection                | High       | Medium   | High                     | Low           |
| T5: Malicious code component        | Low        | High     | High                     | Low           |
| T6: Supply chain attack             | Medium     | Medium   | Medium                   | Medium        |
| T7: Event store corruption          | Low        | High     | High                     | Low           |
| T8: Macro execution attack          | Low        | High     | High                     | Low           |
| T9: Unauthorized channel activation | Medium     | High     | High                     | Low           |
| T10: Entity privilege escalation    | Medium     | High     | High                     | Low           |
| T11: Identity spoofing              | Medium     | High     | Medium                   | Medium        |
| T12: Malicious identity merge       | Low        | High     | High                     | Low           |
| T13: Control channel hijacking      | Low        | High     | High                     | Low           |
| T14: Unauthorized sensor control    | Low        | High     | High                     | Low           |
| T15: Contextual metadata leakage    | Low        | Medium   | High                     | Low           |
| T16: Admin delegate overtake        | Low        | Critical | High                     | Low           |

---

**Related:** [SECURITY.md](SECURITY.md) for the full security specification. [TECHNICAL_CONCEPT.md](../TECHNICAL_CONCEPT.md#15-security-and-privacy-considerations) for the original security considerations.
