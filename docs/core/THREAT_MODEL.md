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

## Trust Boundary Summary

```
┌─────────────────────────────────────────────────────────────┐
│  Untrusted                                                   │
│  ┌─────────────────┐  ┌─────────────────┐                   │
│  │  LLM Provider   │  │  External APIs  │                   │
│  │  (compromised)  │  │  (compromised)  │                   │
│  └────────┬────────┘  └────────┬────────┘                   │
│           │ normalized         │ filtered                   │
│           │ requests only      │ views only                 │
├───────────┼────────────────────┼────────────────────────────┤
│  Semi-Trusted                  │                            │
│  ┌─────────┴──────────────────┴──────────┐                 │
│  │          Edge Node (PEN)              │                 │
│  │  (physical access possible)           │                 │
│  │  ↓ structured events only             │                 │
├──┼───────────────────────────────────────┼─────────────────┤
│  │  Trusted                              │                 │
│  │  ┌───────────────────────────────────┴───────────────┐  │
│  │  │              Homelab (Trusted Core)               │  │
│  │  │  Event Store + Kernel + Memory + Knowledge        │  │
│  │  │  All execution validated, all events hashed       │  │
│  │  └───────────────────────────────────────────────────┘  │
└──┼─────────────────────────────────────────────────────────┘
   │
   ▼
Data flows: untrusted → semi-trusted → trusted (one-way validation)
```

## Security Posture Summary

| Threat                       | Likelihood | Impact | Mitigation Effectiveness | Residual Risk |
| ---------------------------- | ---------- | ------ | ------------------------ | ------------- |
| T1: Network interception     | Medium     | High   | High                     | Low           |
| T2: Physical tampering       | Low        | High   | Medium                   | Medium        |
| T3: LLM compromise           | Medium     | High   | High                     | Low           |
| T4: Prompt injection         | High       | Medium | High                     | Low           |
| T5: Malicious code component | Low        | High   | High                     | Low           |
| T6: Supply chain attack      | Medium     | Medium | Medium                   | Medium        |
| T7: Event store corruption   | Low        | High   | High                     | Low           |
| T8: Macro execution attack   | Low        | High   | High                     | Low           |

---

**Related:** [SECURITY.md](SECURITY.md) for the full security specification. [TECHNICAL_CONCEPT.md](../TECHNICAL_CONCEPT.md#15-security-and-privacy-considerations) for the original security considerations.
