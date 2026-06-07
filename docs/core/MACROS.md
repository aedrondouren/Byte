# Macro System

> Expands on the Macro System section of [TECHNICAL_CONCEPT.md](../TECHNICAL_CONCEPT.md#91-macro-system).

## What Macros Are

Macros are compiled execution subgraphs that capture repeated behavior as deterministic conditional logic and tool calls, annotated with Reasoning Hints.

A macro is not a reasoning engine. It does not yield, pause, or resume into general reasoning. It executes to completion as a deterministic piece of the execution graph.

## Reasoning Hints

A Reasoning Hint is a distilled textual annotation written by the model during macro compilation. It captures the outcome of a reasoning step at a branch point, not the full chain-of-thought that produced it.

Example from a compiled macro:

```
If budget >= item.cost:
  Hint: "Budget allows for this item, proceeding with purchase"
  Tool: purchase(item)

If budget < item.cost:
  Hint: "Budget insufficient for this item, skipping"
  Tool: skip(item)
```

Hints serve three purposes:

1. **Provenance** — when auditing a macro execution, the hint chain explains why each branch was taken without reconstructing the original reasoning.
2. **Context injection** — hints and tool results from recent macro execution form a sliding window passed to the RPU when the kernel invokes it for new reasoning.
3. **Future learning** — the hint chain gives future macro discovery passes semantic context to work with, not just tool-call sequences.

### Properties

**Distilled** — a hint captures the outcome of reasoning, not the reasoning process itself. It is a concise explanation of why a decision was made.

**Parameterized** — concrete values from the original execution become variables, allowing the same macro to generalize across different situations.

```
If distance <= {max_distance}:
  Hint: "Target within {max_distance} km, using local search"
  Tool: search_local(target)
```

**Immutable** — once a macro is validated, its hints become part of the macro's commit range. They are not regenerated during execution.

**Expandable** — a macro always preserves the link to its originating events. No behavior is hidden behind abstraction.

## Discovery Mechanisms

### Sliding Window Mining

The system scans execution traces through a sliding time window (configurable, default 24 hours). Within each window, it identifies repeated sequences of tool calls, RPU invocations, and scheduler decisions. The window slides forward in increments (default 1 hour) to catch patterns that span window boundaries.

Normalization is applied before comparison: variable identifiers are replaced with type placeholders, timestamps are converted to relative offsets, and confidence scores are bucketed into ranges. This prevents superficial differences from masking structural similarity.

### Subgraph Matching

Execution traces are represented as directed graphs where nodes are actions (tool calls, RPU invocations, state transitions) and edges are causal dependencies. The mining algorithm searches for frequently occurring subgraphs using a variant of the gSpan algorithm adapted for execution traces.

Key adaptations:

- Nodes carry typed payloads (tool name, RPU function, state update type) rather than simple labels
- Edges carry temporal offsets and priority context
- Subgraph frequency is weighted by recency — patterns that occurred recently are more valuable than patterns from months ago
- Semantic equivalence is checked at the payload level, not just the graph structure

### Pattern Normalization and Hint Extraction

Before a pattern can be promoted, it must be normalized into a reusable form:

1. Concrete values are replaced with parameters
2. Fixed tool references are generalized to service abstractions
3. Hardcoded thresholds become configurable parameters
4. Branch conditions are extracted as decision points
5. The model distills the reasoning at each branch point into a Reasoning Hint

This normalization is assisted by LLM analysis of the trace patterns, but the resulting macro is validated deterministically.

## Context Window Integration

When a macro completes and the kernel decides to invoke the RPU for new reasoning, the RPU receives a sliding window of recent execution context. This window contains the most recent hint/tool-result pairs from the macro's execution:

```typescript
interface RPURequest {
  function: string;
  objective: string;
  personality: PersonalityState;
  worldState: WorldState;
  context: ContextProjection;
  recentExecution: {
    hint: string;
    toolResult: unknown;
  }[];
  taskState?: TaskState;
  previousArtifacts?: Artifact[];
}
```

This gives the RPU local awareness of what just happened and why, alongside targeted memories from the memory graph and validated facts from the knowledge graph. The window is bounded so it doesn't inflate context size unnecessarily — only recent decisions matter for immediate next-step reasoning.

## Validation Test Design

A proposed macro is validated against historical execution data. The test process:

1. **Replay** — the macro is executed against the same input traces that originally produced the pattern
2. **Equivalence check** — the macro's output is compared to the original execution output at the semantic level (not byte-for-byte, but functionally equivalent)
3. **Edge case testing** — the macro is tested against variations of the original traces: missing inputs, different tool responses, priority conflicts
4. **Performance measurement** — the macro's resource consumption is measured against the original execution to confirm cost savings

A macro passes validation only when it produces semantically equivalent results across all test traces and demonstrates measurable efficiency gains.

## Demotion Mechanics

A macro can be demoted for several reasons:

- **Usage decay** — the macro is not invoked for a configurable period (default 30 days)
- **Primitive change** — an underlying tool or service that the macro depends on has changed its contract
- **Failure rate increase** — the macro's success rate drops below a threshold due to environmental changes
- **Better replacement** — a more general macro subsumes the functionality of the existing macro

Demotion is reversible. A demoted macro remains in the macro graph with a `demoted` status. It can be re-promoted if conditions change. The full provenance chain — from original trace events through macro proposal, validation, compilation, and demotion — is preserved.

Demoted macros are excluded from the scheduler's macro lookup but remain accessible for audit and re-evaluation.
