# Reasoning Processing Unit (RPU)

> This document expands on the Reasoning Processing Unit section of [TECHNICAL_CONCEPT.md](../TECHNICAL_CONCEPT.md).

## Overview

The Reasoning Processing Unit (RPU) is a design pattern that treats a Large Language Model as a specialized reasoning coprocessor rather than as a complete autonomous agent.

Instead of embedding memory, planning, identity, world state, execution management, and communication into a single prompt, these concerns are externalized into dedicated runtime systems. The model becomes a deterministic cognitive transformation engine operating over structured state.

The objective is not to maximize model intelligence, but to minimize the amount of reasoning required to produce intelligent behavior.

---

## Core Principle

Intelligence is not the amount of reasoning performed.

Intelligence is the ability to accumulate structure so that less reasoning is required in the future.

The RPU architecture therefore seeks to transform repeated reasoning into persistent state, projections, plans, memories, and reusable cognitive artifacts.

---

## Design Philosophy

Traditional agent architectures often assume:

```text
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

The RPU model instead assumes:

```text
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

The agent is not the model.

The agent emerges from the interaction between runtime subsystems and reasoning functions.

---

## RPU Responsibilities

The RPU is responsible for:

- Interpretation
- Reasoning
- Synthesis
- Planning
- Summarization
- Reflection
- Communication generation
- Cognitive transformations

The RPU is not responsible for:

- Memory persistence
- Event storage
- Scheduling
- State tracking
- Personality storage
- Tool orchestration
- World modeling
- Execution management

These functions belong to the runtime.

---

## Cognitive Contract

Every inference executes through a structured contract.

### Input

```typescript
interface RPURequest {
  function: string;
  objective: string;

  personality: PersonalityState;
  worldState: WorldState;
  context: ContextProjection;

  taskState?: TaskState;
  previousArtifacts?: Artifact[];
}
```

### Output

```typescript
interface RPUResponse {
  result: unknown;

  summary?: string;
  plan?: Plan;
  memorySuggestions?: MemoryEvent[];
  stateUpdates?: StateUpdate[];
  nextActions?: ActionProposal[];

  metadata?: {
    confidence?: number;
    reasoningMode?: string;
  };
}
```

The RPU produces structured cognitive artifacts rather than raw conversational text.

---

## Planning-First Execution

Long-running tasks follow a planning-first workflow.

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

The plan becomes an observable runtime artifact.

Execution updates the plan state rather than generating arbitrary conversational messages.

This creates deterministic visibility into system progress.

---

## Recap-Based Observability

The runtime maintains execution state independently from the conversation.

Users may request:

- Current status
- Task progress
- Plan state
- Execution recap

The response is generated from runtime projections rather than reconstructed from conversation history.

Example:

```text
Task: Research Topic X

Plan:
✓ Gather Sources
✓ Extract Findings
⟳ Generate Report
□ Final Review
```

This avoids using conversational messages as the source of truth.

---

## Personality as State

Personality is stored as structured data rather than prompt text.

```json
{
  "curiosity": 0.9,
  "humor": 0.4,
  "technicalDepth": 0.95,
  "directness": 0.8
}
```

The runtime owns personality.

The RPU consumes personality projections.

This allows:

- Model replacement
- Personality versioning
- Personality evolution
- Consistent behavior across backends

---

## Event-Sourced Cognition

All runtime actions generate events.

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

The event log becomes the canonical source of truth.

The conversation is merely one projection of runtime state.

---

## World-State Architecture

The context window is not treated as the world.

Instead:

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

The RPU reasons over structured representations of reality rather than reconstructing reality from conversational history.

---

## Model Independence

The RPU abstraction enables model interchangeability.

The runtime defines:

- State
- Contracts
- Events
- Personality
- Memory
- Scheduling

The model only implements reasoning functions.

```text
Runtime
    ↓
RPU Contract
    ↓
Any Compatible Model
```

This allows multiple model classes to participate in the same cognitive runtime.

---

## Cognitive Cache Hierarchy

The architecture can be viewed as a hierarchy of increasingly expensive cognitive operations.

```text
L1 - Active Context
L2 - Structured State
L3 - World Model
L4 - Event History
L5 - Reasoning (RPU)
```

Reasoning is the most expensive resource.

The purpose of the runtime is to convert reasoning into reusable structure and prevent unnecessary recomputation.

---

## Guiding Question

The central design question of the RPU architecture is:

"What can be removed from the reasoning loop?"

Any responsibility that can be represented as deterministic state, stored structure, or runtime infrastructure should be externalized from the model.

The RPU should only perform transformations that genuinely require reasoning.

This allows intelligent behavior to emerge from accumulated structure rather than repeated inference.
