# Reasoning Processing Unit (RPU)

> Expands on the Reasoning Processing Unit section of [TECHNICAL_CONCEPT.md](../TECHNICAL_CONCEPT.md#6-reasoning-processing-unit).
> **Audience:** Technical
> **Prerequisites:** TECHNICAL_CONCEPT.md Sections 1–6, ORCHESTRATION.md
> **Status:** Complete

## Design Philosophy — Visual

Traditional approach (everything in one prompt):

```
Conversation
+
System Prompt
+
Tools
+
Memory Instructions
+
Personality Instructions
=
Agent
```

RPU approach (concerns separated):

```
Runtime State
+
World State
+
Personality State
+
Task Definition
+
Reasoning Function
=
Cognitive Result
```

Agent emerges from the interaction between runtime subsystems and reasoning functions.

## Cognitive Flow Diagrams

### Event-sourced cognition

```text
User Request
      ↓
    Event
      ↓
  Projection
      ↓
    State
      ↓
RPU Invocation
      ↓
  New Events
```

### World-state architecture

```text
Sensors
Memory
Events
Tools
External Systems
        ↓
  World State
        ↓
  Projection
        ↓
    RPU
```

The RPU reasons over structured representations of reality — projections of the event graph and current cognitive artifacts — rather than reconstructing reality from conversational history.

### Model independence

```text
Runtime
    ↓
RPU Contract
    ↓
Any Compatible Model
```

### Cognitive cache hierarchy

```text
L1 - Active Context
L2 - Structured State
L3 - World Model
L4 - Event History
L5 - Reasoning (RPU)
```

## Planning-First Execution — Visual

```text
User Request
      ↓
Acknowledgement
      ↓
Plan Generation
      ↓
Execution
      ↓
Recap
      ↓
Conversation Response
```

## Recap Example

```text
Task: Research Topic X

Plan:
✓ Gather Sources
✓ Extract Findings
⟳ Generate Report
□ Final Review
```

Status markers: ✓ complete, ⟳ in progress, □ pending.

## RPU Contract Specification

### Request Schema

Every RPU invocation follows a structured contract. The kernel constructs the request from world-state projections; the RPU returns structured results.

```typescript
interface RPURequest {
	function: string; // The reasoning function to execute
	objective: string; // What success looks like
	personality: PersonalityState;
	worldState: WorldState;
	context: ContextProjection; // Filtered by retrieval pipeline (RETRIEVAL.md)
	capturedPlan?: Plan; // From macro execution, present only if the macro captured a plan artifact (MACROS.md)
	taskState?: TaskState;
	previousArtifacts?: Artifact[];
	recentExecution?: {
		// From macro execution (MACROS.md)
		hint: string;
		toolResult: unknown;
	}[];
}

interface ContextProjection {
	memories: MemoryEvent[]; // Filtered by entity, channel, domain, privacy, relevance
	knowledge: KnowledgeEntry[]; // Dual-access: factual content + scoped contextual metadata
	entities: EntityDefinition[]; // loaded from Entity Graph
	entityStates: EntityState[]; // projected from event stream
	channel: ChannelPolicy; // Active channel constraints
	activeEntity: EntityDefinition; // current version from Entity Graph
	activeEntityState: EntityState; // projected state for active entity
	permissionSummary: {
		accessibleDomains: string[]; // Domains this entity can access
		privacyCeiling: string; // Maximum privacy level
		controlAuthority: boolean; // Can send kernel control signals
	};
}
```

### Response Schema

```typescript
interface RPUResponse {
	result: unknown; // Primary output of the reasoning function
	summary?: string; // Human-readable summary
	plan?: Plan; // Proposed execution plan
	memorySuggestions?: MemoryEvent[];
	stateUpdates?: StateUpdate[];
	nextActions?: ActionProposal[];
	metadata?: {
		confidence?: number; // 0.0–1.0, calibrated against actual correctness
		reasoningMode?: string; // e.g., "analytical", "creative", "recall"
		modelVersion?: string; // For debugging probabilistic projections
		tokenUsage?: {
			// For cost tracking
			input: number;
			output: number;
		};
	};
}
```

### Latency Budgets

Expected response times per function type:

| Function Type                | Expected Latency | Timeout     | Priority Tier |
| ---------------------------- | ---------------- | ----------- | ------------- |
| Situation model generation   | 2–5 seconds      | 10 seconds  | Interactive   |
| Plan generation              | 5–15 seconds     | 30 seconds  | Interactive   |
| Summarization                | 10–30 seconds    | 60 seconds  | Background    |
| Knowledge validation         | 5–20 seconds     | 30 seconds  | Background    |
| Conversation response        | 1–3 seconds      | 5 seconds   | Interactive   |
| Macro compilation assistance | 15–60 seconds    | 120 seconds | Background    |

These budgets assume a modern LLM (GPT-4-class or equivalent). Smaller models may have different latency profiles; the kernel adapts timeouts based on observed model performance.

## Error Handling Model

### Malformed Output

When the RPU returns output that does not conform to the expected schema:

1. **Validation failure is logged** as an event in the execution graph with the raw output preserved for debugging.
2. **The kernel retries** with a corrected prompt that includes the validation error message (max 2 retries).
3. **If retries fail**, the chain is marked as failed with an "RPU validation error" status. The kernel falls back to rule-based heuristics if available.
4. **The model version and request context** are recorded for post-mortem analysis.

### Timeout

When the RPU does not respond within the allocated timeout:

1. **The request is cancelled** at the provider level (if the provider supports cancellation).
2. **The chain is parked** and can be resumed when the RPU becomes available.
3. **If the chain is time-sensitive** (interactive priority), the kernel falls back to a cached or heuristic response.
4. **Timeout events** are recorded and contribute to model reliability metrics.

### Model Failure

When the RPU provider is unavailable or returns an error:

1. **The LLM Runtime adapter** (TECHNICAL_CONCEPT.md Section 6.9) attempts to route to an alternative provider if configured.
2. **If no alternative is available**, the chain is parked with a "model unavailable" status.
3. **Critical chains** that require immediate reasoning escalate to the user with a "reasoning unavailable" notification.
4. **The failure is recorded** and contributes to provider reliability metrics.

### Confidence Score Semantics

The RPU returns a confidence score (0.0–1.0) with each response. This score represents the model's estimated probability that the response is correct and complete.

| Confidence Range | Interpretation      | Kernel Action                                              |
| ---------------- | ------------------- | ---------------------------------------------------------- |
| 0.9–1.0          | High confidence     | Accept and proceed                                         |
| 0.7–0.9          | Moderate confidence | Accept but flag for review; may request additional context |
| 0.5–0.7          | Low confidence      | Request additional context or fall back to heuristic       |
| Below 0.5        | Very low confidence | Reject; fall back to heuristic or escalate to user         |

Confidence scores are calibrated over time by comparing them against actual correctness (determined by user feedback, validation tests, or downstream success). The calibration curve is stored as a projection and used to adjust raw confidence scores before kernel decision-making.

## Model Versioning Strategy

### Version Pinning

Each RPU invocation records the model version used. This enables:

- **Replayability:** Probabilistic projections can be replayed with the same model version to reproduce results.
- **A/B testing:** Different model versions can be compared on the same tasks.
- **Rollback:** If a model version performs poorly, the kernel can pin to a previous version.

### Version Format

Model versions are identified by a structured string: `provider:model:version` (e.g., `openai:gpt-4:2024-04-09`). The LLM Runtime normalizes provider-specific version formats into this canonical form.

### Model Swap Procedure

When switching to a new model:

1. **The new model is tested** against a benchmark suite of canonical tasks.
2. **Confidence calibration is rebuilt** for the new model (confidence scores are model-specific).
3. **Latency budgets are adjusted** based on observed performance.
4. **The swap is recorded** as an event; all subsequent invocations use the new version.
5. **The old version remains available** for replay and comparison.

## Fallback Behavior

When the RPU is unavailable, the system degrades gracefully:

| Component                  | Fallback                                       |
| -------------------------- | ---------------------------------------------- |
| Situation model generation | Rule-based heuristics from perception features |
| Plan generation            | Template-based plans from task type            |
| Summarization              | Deterministic aggregation (no LLM)             |
| Knowledge validation       | Statistical corroboration (no LLM)             |
| Conversation response      | Pre-built response templates                   |
| Macro compilation          | Deferred until RPU available                   |

The fallback behavior is less capable but maintains system safety and core functionality. This is consistent with the "graceful degradation is mandatory" invariant.

## Rate Limiting and Quota Enforcement

The kernel enforces RPU usage quotas:

- **Per-chain quota:** Each execution chain has a token budget. If the budget is exceeded, the chain is paused and requires user approval to continue.
- **Per-time-window quota:** The system tracks total RPU usage over configurable time windows (default: 1 hour). If the window quota is exceeded, non-critical chains are parked until the window resets.
- **Cost-aware scheduling:** The scheduler considers estimated token cost when allocating RPU capacity. High-cost functions are scheduled during low-demand periods when possible.

## Closing Principle

Intelligent behavior emerges from accumulated structure rather than repeated inference.

---

**Related:** [TECHNICAL_CONCEPT.md](../TECHNICAL_CONCEPT.md#6-reasoning-processing-unit) for the original specification. [ORCHESTRATION.md](ORCHESTRATION.md#core-primitives--semantic-specifications) for the execution primitives the RPU operates within. [MACROS.md](MACROS.md#context-window-integration) for how RPU-assisted macro compilation works.
