# Design Decisions

> Rationale behind key architectural choices in the B.Y.T.E. architecture.
> **Audience:** Technical
> **Prerequisites:** TECHNICAL_CONCEPT.md (full)
> **Status:** Complete

This document explains why the architecture makes the choices it does. Each decision includes the alternatives considered and the reasoning for the chosen approach.

---

## Why "Git, Not Blockchain"

**Decision:** The world-state graph uses a Git-like model (content-addressed, immutable, DAG-structured) rather than a blockchain model.

**Alternatives considered:**

- Blockchain with consensus (Bitcoin/Ethereum model)
- Append-only log with centralized authority (Kafka model)
- Mutable database with audit log (traditional model)

**Reasoning:**

- The goal is tamper-evident history, causal lineage, replayability, auditability, and reproducibility — not distributed consensus. There is a single trusted authority (the homelab), so consensus mechanisms are unnecessary overhead.
- Git's DAG model naturally represents causal relationships between events. Each event references its parents, creating a verifiable chain of causality.
- Content-addressing (identifying events by their hash) ensures immutability without requiring a consensus protocol.
- The blockchain analogy breaks down on branching/merging semantics. Git's branch model (lightweight, mergeable, with conflict detection) is closer to the execution chain model than blockchain's linear chain model.

**Where the analogy breaks down:**

- Git is designed for human-authored code with infrequent merges. The world-state graph is machine-authored with frequent concurrent branches. Merge semantics differ.
- Git's conflict resolution is manual. The world-state graph needs automated conflict resolution (deferred to Phase 6).
- Git stores full file contents. The world-state graph stores structured events with projections.

---

## Why Two Stores, Seven Graphs

**Decision:** The world-state graphs (perception, execution, memory, knowledge) share an append-only event store. The artifact graphs (macro, skill, registry) use a separate versioned artifact store. Cross-store references use content hashes in events pointing to artifact ID+version — no unified index needed.

**Alternatives considered:**

- Single event store for all seven graphs (original design)
- Separate stores for each graph
- Unified index mapping both stores

**Reasoning:**

- **Different data models.** World-state graphs are projections of an append-only event stream — state is derived by replaying events. Artifact graphs are versioned entities with semantic versioning and lifecycle management — state is the latest version per artifact. Conflating these models creates conceptual and practical problems.
- **Different query patterns.** World-state queries are temporal and causal ("what happened between T1 and T2?"). Artifact queries are identity-based ("what is the current version of macro X?"). Separate stores keep queries simple and indexing efficient.
- **Different lifecycle.** Events are immutable and never change. Artifacts have lifecycle states (created, promoted, demoted, superseded, deprecated) and new versions can be created. An event store is not designed for version management.
- **Different retention.** Events can be archived to cold storage after a time window. Artifacts must remain accessible as long as any reference points to them — retention is by reference, not by time.
- **Implementation reality.** You'd use different storage backends: an event log (time-series DB or append-only log) for world-state, and a content-addressed store (Git-like or object store with metadata) for artifacts.

**Trade-off:** Cross-store references require two lookups instead of one. However, this is O(1) + O(1) with proper indexing — the event stores the artifact ID+version, and the artifact store lookup is a direct hash. The cost is negligible compared to the clarity gained by separating the models.

**Where the original design broke down:** The "everything is an event" model was a useful simplification during early design but doesn't hold under scrutiny. A macro v2 is not an "event" — it's a new version of an artifact. The event that _created_ macro v2 is in the world-state store; the macro artifact itself is not. Similarly, a skill definition is not an event — it's a versioned entity that generates events when executed.

---

## Why Seven Graphs vs. One Graph with Tags

**Decision:** The world-state is conceptually split into four logical graphs (perception, execution, memory, knowledge) within the event store, and three logical graphs (macro, skill, registry) within the artifact store.

**Alternatives considered:**

- One graph with type tags on all events
- Seven separate physical databases
- One graph with separate indexes per domain

**Reasoning:**

- **Conceptual separation keeps queries simple and indexing efficient.** A query for "all memories relevant to the current situation" should not scan perception events. A query for "current version of macro X" should not scan the event log.
- **Physical separation by data model** ensures that each store uses the indexing strategy appropriate to its data type — temporal/causal for events, identity/version for artifacts.
- **Each graph maintains its own indexing strategy and retention policies.** Perception events may be retained for days; knowledge entries may be retained indefinitely; artifact versions are retained as long as referenced.
- **The "two stores, seven graphs" model** provides the best balance of conceptual clarity and implementation simplicity.

---

## Why TypeScript + Effect as the Primitive Vocabulary

**Decision:** The semantic orchestration layer uses TypeScript + Effect as the reference implementation and common primitive vocabulary.

**Alternatives considered:**

- Rust (strong type safety, but different concurrency model)
- Go (simple concurrency, but lacks Effect's error handling model)
- Custom DSL (full control, but high implementation cost)
- Python (ecosystem, but lacks type safety for semantic guarantees)

**Reasoning:**

- Effect.ts provides a comprehensive, well-tested set of semantic primitives (run, fork, parallel, race, scope, acquire/release, etc.) that map closely to the orchestration layer's requirements.
- TypeScript's type system provides compile-time guarantees about primitive composition.
- Effect's layer/context system provides the dependency injection model needed for Tool Services and Application Services.
- TypeScript + Effect transpiles nearly 1:1 to Go (REGISTRY.md), enabling cross-language composition.
- The choice is about semantics, not implementation. The primitives are language-agnostic; TypeScript + Effect is the reference.

---

## Why Event-Sourced vs. State-Based

**Decision:** The world-state is event-sourced (append-only event log, state as projection) rather than state-based (mutable database with current state).

**Alternatives considered:**

- Mutable database with audit log
- Event sourcing with periodic snapshots as primary state
- CRDT-based eventual consistency

**Reasoning:**

- Event sourcing provides full provenance. Every action is traceable to its origin. This is essential for debugging, auditability, and system improvement.
- State as projection means the system can always rebuild current state from history. Checkpoints accelerate reconstruction but do not replace history.
- The append-only model simplifies concurrency. There are no write conflicts because events are never modified.
- The trade-off is storage growth and query complexity. The summarization pipeline (TECHNICAL_CONCEPT.md Section 8) addresses storage by compressing events into memories and knowledge. Indexing strategies (GRAPH.md) address query complexity.

---

## Why RPU as Coprocessor vs. Autonomous Agent

**Decision:** The LLM is a reasoning coprocessor (RPU) invoked by the kernel with structured contracts, not an autonomous agent that controls the system.

**Alternatives considered:**

- Autonomous agent (LLM controls execution, tool use, and memory)
- ReAct-style agent (LLM interleaves reasoning and action)
- Tool-use agent (LLM calls tools through a defined interface)

**Reasoning:**

- Separation of concerns: the kernel manages execution, scheduling, and safety. The RPU performs transformations that genuinely require reasoning. This prevents the LLM from making irreversible decisions without validation.
- Model independence: the RPU contract is model-agnostic. Swapping models does not change system behavior.
- Auditability: every RPU invocation is a structured event with input, output, and metadata. This is easier to audit than an autonomous agent's free-form reasoning.
- Cost control: the kernel decides when to invoke the RPU and with what context. This prevents unnecessary inference calls.
- The "AI proposes intent; the kernel executes deterministically" invariant (TECHNICAL_CONCEPT.md Section 1.4) ensures that the system remains safe even if the LLM produces incorrect or malicious output.

---

## Why Projection vs. Channel Distinction

**Decision:** The runtime distinguishes between projections (graph → graph transformations) and channels (graph → external surface transformations).

**Alternatives considered:**

- Single transformation model (no distinction)
- Pipeline model (all transformations are sequential)
- Event-driven model (all transformations are event handlers)

**Reasoning:**

- Projections build runtime knowledge; channels consume it. This distinction prevents channels from creating a second source of truth.
- Projections are durable; channels come and go. The same projection graph can serve any number of channels without conceptual changes.
- The decision rules are actionable: "If a transformation doesn't produce durable runtime knowledge, it's a channel, not a projection." This prevents architectural drift.
- Channels are leaves in the transformation pipeline. Nothing projects further from a channel. This prevents circular dependencies between external surfaces and internal state.

---

## Why Defer Known Gaps

Several gaps are acknowledged but not resolved (KNOWN_GAPS.md). The deferral rationale:

- **Macro discovery:** The core thesis holds through memory and knowledge compression even if macro discovery fails. Macro discovery is an optimization, not a foundation. Deferred until Phases 1–4 validate the thesis.
- **Knowledge extraction rules:** Will emerge from implementation and testing rather than theoretical design. The validation pipeline (corroboration, contradiction detection, confidence scoring) provides a framework; the specific rules need empirical tuning.
- **Distributed conflict resolution:** The architecture is single-node with edge-cloud separation. Multi-node conflict resolution is a Phase 6 problem. CRDTs, vector clocks, or merge policies will be evaluated when the need arises.
- **EEG and experimental modalities:** Not required for the core thesis. Included to explore the concept's boundaries, not as foundational requirements.

## Why Temporal Intent Is a Cron Subsystem

**Decision:** Temporal intent operates as a deterministic cron-like scheduling subsystem managed by the kernel, not as an RPU-driven reasoning function. The RPU identifies patterns and suggests schedules; the user approves; the kernel fires them automatically.

**Alternatives considered:**

- RPU-generated intent on every cycle (AI decides when to act)
- Fully autonomous temporal reasoning (no user approval)
- Manual configuration only (no pattern detection)

**Reasoning:**

- **Deterministic scheduling is safer and auditable.** A cron entry is a concrete, inspectable schedule. The user can see what is scheduled, when it fires, and what it does. An RPU that decides on the fly whether to act is opaque and harder to debug.
- **No RPU capacity consumed at fire time.** The schedule itself is the proposal. When a cron entry fires, the kernel generates an intent event directly — no model invocation needed. This aligns with the thesis of reducing reasoning cost.
- **User approval provides a consent gate.** The RPU suggests, the user decides. This prevents the system from automating behaviors the user doesn't want, while still enabling pattern discovery.
- **Destructive/non-destructive tagging at the tool service level** provides a safety layer without requiring the user to understand internals. Read-only tools (weather fetch, calendar lookup) can auto-execute above a confidence threshold. State-changing tools (send email, modify files) always require per-execution approval.
- **Confidence feedback loop** enables self-correction. User dismissal reduces confidence; repeated dismissals demote the cron entry. The system learns from interaction, not from configuration.

**Trade-off:** The user must approve each pattern suggestion. This adds friction compared to fully autonomous automation but prevents unwanted behavior. The confidence threshold for non-destructive auto-execution is configurable, allowing users to tune the balance between convenience and control.

## Why Skills in Existing Harness Format

**Decision:** Skills use the existing harness format (instruction files + tool references) that users already have from their current AI setups.

**Alternatives considered:**

- New declarative skill format (structured JSON/YAML)
- Imperative Effect-style function definitions
- Prompt-only skills with no tool declarations

**Reasoning:**

- **Zero learning curve.** Users can drop their existing skills into the system and get optimization automatically. No migration, no re-authoring.
- **Execution optimization applies universally.** The system optimizes any skill regardless of its internal structure because optimization happens at the execution trace level, not the instruction level. The RPU executes the skill's instructions; the discovery pipeline analyzes the resulting traces.
- **Ecosystem compatibility.** The existing skill ecosystem (community repositories, third-party authors, tool integrations) becomes immediately available. The system does not need to build its own skill catalog from scratch.
- **Format independence.** Since skills are executed through the RPU contract, the system does not need to parse or interpret skill instructions. This means the skill format can evolve independently of the runtime.

**Trade-off:** While skills are text files and can be statically analyzed (tool enumeration, keyword detection, instruction parsing), the primary optimization path is trace-based — because execution traces capture actual behavior, not intended behavior. Static analysis may supplement trace-based optimization in future phases (e.g., pre-computing tool access maps, detecting potential risk patterns for the security audit). The choice to prioritize trace-based optimization is not a limitation of static analysis capability, but a design decision about what produces reliable optimization signals.

## Why Hierarchical Macros

**Decision:** Macros can reference other macros as child subgraph components, enabling composition and reuse.

**Alternatives considered:**

- Flat macros only (no composition)
- Function-call-style macro composition (runtime call stack)
- DAG subgraph reuse (chosen approach)

**Reasoning:**

- **DAG subgraph reuse matches the execution model.** The execution graph is a DAG by construction. Hierarchical macros are just higher-level views of the same DAG structure — logical groupings of subgraph references, not runtime calls.
- **Incremental compilation.** Leaf macros are discovered first (higher frequency). Parent macros are discovered later as composition patterns emerge. The system does not need to discover the entire hierarchy at once.
- **Better discovery partitioning.** Small, focused patterns are easier to discover and validate than large, complex ones. Hierarchical composition lets the system build sophisticated behavior from simpler, well-tested components.
- **No cycles by construction.** The execution model is event-driven, not call-driven. There is no call stack, no recursion. Macros are compiled subgraphs that are always expandable to their full event ranges. Circular references are impossible.
- **Always expandable.** Like folders in a tree view, macros can be recursively expanded to reveal the complete execution history. Tracing is a UI concern, not an architectural constraint. No depth limits are needed.

**Trade-off:** Hierarchical macros add complexity to the discovery pipeline (composition detection, parameter binding, error propagation). This complexity is managed by separating leaf macro discovery from composition discovery into two offline passes.

## Why Skills Are Optional

**Decision:** Skills are an optional acceleration layer. The system works without any skills installed, discovering patterns organically from its own execution. Users may install zero, one, or many skills at any time.

**Alternatives considered:**

- Mandatory skill catalog (system ships with required skills)
- Skills as the primary behavior model (no organic discovery)
- Skills as the only seed mechanism

**Reasoning:**

- **User autonomy.** The user decides what behaviors to pre-define. Some users will want a rich skill setup; others will prefer the system to learn organically.
- **System works without them.** The compression pipeline operates on general execution traces regardless of skill presence. Skills accelerate the path to compressed behavior but don't change the destination.
- **Acquire over time.** Users can install skills at any point — during setup, or later as needs emerge. The system may also suggest skills based on observed behavior patterns.
- **No lock-in.** Users who install skills and later remove them don't lose system functionality. The system continues with organic pattern discovery.

**Trade-off:** Without skills, the cold-start period is longer. The system must discover all patterns organically, which takes more execution history before macros appear. This is accepted as the cost of user autonomy.

## Why Macros Capture Whatever Repeats

**Decision:** A macro compiles the repeating subgraph as detected by the discovery pipeline. It does not mandate a predetermined structure. The captured content may include tool calls, reasoning hints, plan artifacts, or any combination — whatever repeated.

**Alternatives considered:**

- Fixed macro structure (always: plan + hints + tools)
- Separate macro types for different content levels
- Mandate plan artifact capture for all macros

**Reasoning:**

- **Faithful compilation.** The macro is a compression of observed behavior, not a template. If the repetition was at the tool-call level, the macro captures tool calls. If the full planning-first chain repeated, the macro captures everything including the plan artifact.
- **Generality.** A single macro model handles all cases — from the simplest sequential tool call to the most complex planning-first chain with branching and plan artifacts. No need for separate macro types.
- **Plan artifacts are contingent.** When present, they provide observability (user sees intent at plan level), escalation context (RPU receives plan when reasoning is needed), and intent trace (preserves original intent). When absent, the macro still functions — it just lacks those benefits.
- **No over-engineering.** Mandating plan artifact capture for all macros would require generating plans for patterns that never involved planning, which would be artificial and potentially misleading.

**Trade-off:** Macros have variable structure, which makes the macro graph more heterogeneous. This is managed through the captured content spectrum documentation and optional fields in the macro schema.

## Why Open Skill Ecosystem with Advisory Audit

**Decision:** Skills come from any source. The user decides what to trust. The system provides an advisory security audit that raises events but does not block execution.

**Alternatives considered:**

- Curated skill marketplace (system-approved only)
- Mandatory security gate (skills blocked until audit passes)
- Open ecosystem with no audit

**Reasoning:**

- **User autonomy.** The user is the ultimate authority over what runs on their system. A curated marketplace or mandatory gate would impose the system's judgment over the user's.
- **Ecosystem growth.** An open ecosystem encourages skill authoring and sharing. A curated system creates friction and limits the available skill catalog.
- **Advisory audit as middle ground.** The security audit provides value (risk detection, user awareness) without overreach (blocking execution). The user receives information and decides.
- **Primary defense is architectural.** Even a malicious skill cannot execute arbitrary actions. The RPU contract, kernel validation, and tool authorization provide the primary defense. The audit is a secondary, advisory layer.

**Trade-off:** The user bears the responsibility of evaluating skills. The advisory audit helps but does not eliminate this burden. False positives and false negatives in the audit are inevitable. This is accepted as the cost of user autonomy and ecosystem openness.

## Why Admin-First Trust

**Decision:** The Admin entity exists from system bootstrap, pre-linked to the primary webUI. All other entities start untrusted. Trust is explicitly granted, never automatic.

**Alternatives considered:**

- Automatic trust based on behavior patterns
- First-connected entity becomes Admin
- Multi-admin consensus model

**Reasoning:**

- **Security.** A single, irrevocable root of trust simplifies the security model. There is no ambiguity about who has ultimate authority.
- **Bootstrap simplicity.** Admin exists from day one — no election or discovery process needed. The primary webUI is the trusted bootstrap channel.
- **Auditability.** All trust changes flow from Admin through explicit actions. The full delegation chain is traceable.
- **User autonomy.** The system owner controls who has access. No entity gains access through behavior alone.

**Trade-off:** The Admin is a single point of trust. If Admin credentials are compromised, the system is compromised. This is mitigated by the control channel flag (secondary channels cannot send control signals even with Admin identity) and by channel-level authentication.

## Why Discord-Style Permissions

**Decision:** Permissions follow a Discord-style hierarchy: Admin (irrevocable) -> admin_delegate (can manage, cannot supersede) -> trusted_user -> sub_user. No delegation chain can produce permissions equal to or greater than Admin.

**Alternatives considered:**

- Flat permission model (all-or-nothing)
- Capability-based model (individual permissions, no hierarchy)
- Role-based access control (RBAC) with arbitrary role creation

**Reasoning:**

- **Familiar model.** Discord-style permissions are widely understood. Admin delegates manage day-to-day operations while the Admin retains ultimate authority.
- **Admin ceiling invariant.** The irrevocable Admin prevents privilege escalation attacks through delegation chains.
- **Graduated trust.** Different entities need different levels of access. A family member needs more access than a guest; a trusted collaborator needs more than a casual user.
- **Simplicity over flexibility.** RBAC offers more flexibility but adds complexity. The Discord-style hierarchy covers the most common use cases without requiring role configuration.

**Trade-off:** Less flexible than RBAC. Unusual permission configurations may not fit the hierarchy. This is accepted in favor of simplicity and security.

## Why Control Channel Flag

**Decision:** Channels are not control channels by default, even for Admin-connected channels. A channel must be explicitly enabled to send kernel control signals.

**Alternatives considered:**

- All Admin-connected channels have control authority
- Control signals require per-signal authentication
- Separate control channel surface (dedicated admin interface)

**Reasoning:**

- **Defense in depth.** Even if an Admin session is compromised on a secondary channel, the attacker cannot send kernel control signals.
- **Explicit intent.** Enabling control authority is a deliberate administrative action, not an implicit consequence of Admin identity.
- **Channel diversity.** A channel used for casual interaction (e.g., a Discord channel) should not have control authority even if the Admin uses it. The control channel flag separates identity from authority.

**Trade-off:** Admin must explicitly enable control on each channel they want to use for system management. This adds friction but prevents accidental or malicious control signal injection.

## Why Dual-Access Knowledge

**Decision:** Knowledge entries carry two access levels: factual content (globally accessible when domain permissions allow) and contextual metadata (scoped to the originating relationship).

**Alternatives considered:**

- Knowledge is either fully accessible or fully restricted
- All knowledge is scoped to originating relationship
- All knowledge is globally accessible

**Reasoning:**

- **Knowledge propagation.** Useful facts should propagate beyond their originating relationship. If Admin learns that "event sourcing is useful," this fact should be available to sub-users working on technical tasks.
- **Contextual protection.** The fact that Admin taught this, under what relationship, and in what context is private information. This metadata should not leak to entities outside the originating relationship.
- **Privacy preservation.** The dual-access model allows factual knowledge to become globally accessible while relationship and provenance context remain scoped.

**Trade-off:** Implementation complexity. The retrieval pipeline must consistently strip contextual metadata for non-originating entities. A single implementation error could leak sensitive relationship information.

## Why Immutable History with Invalidated Derivatives

**Decision:** Memories and events are never modified. Knowledge and macros are invalidated when source entity permissions change.

**Alternatives considered:**

- Delete memories when entity is demoted
- Modify memories to remove entity references
- Keep all derivatives regardless of permission changes

**Reasoning:**

- **Provenance integrity.** Memories and events are the shared substrate of the runtime. Modifying them breaks the provenance chain and undermines the tamper-evident history.
- **Permission respect.** Derived artifacts (knowledge, macros) must respect current trust boundaries. If an entity is demoted, artifacts derived from their execution should be invalidated.
- **Reversibility.** If an entity is re-elevated, the immutable history is still available for re-derivation. Deleting memories would be irreversible.

**Trade-off:** Storage growth. The immutable history grows indefinitely. The summarization pipeline and retention policies address this by archiving old events to cold storage while keeping summaries in hot storage.

---

---

**Related:** [KNOWN_GAPS.md](KNOWN_GAPS.md) for the specific gaps that remain unresolved. [TECHNICAL_CONCEPT.md](../TECHNICAL_CONCEPT.md#14-core-invariants) for the core invariants that drive these decisions.
