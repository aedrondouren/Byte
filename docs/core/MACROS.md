# Macro System

> This document expands on the Macro System section (Section 8.1) of [TECHNICAL_CONCEPT.md](../TECHNICAL_CONCEPT.md).

## What Macros Are

Macros are Effect-style function definitions that compose Tool Services through semantic orchestration primitives. Individual tool calls are exposed as service methods; macros combine them with conditional logic to capture reasoning and tool usage patterns.

Macros use Tool Services only — they never use Application Services.

Macros are runtime-only — they execute in the TypeScript/Effect runtime and are not transpiled.

## Discovery → Validation → Compilation → Demotion

Repeated execution patterns are detected through sliding window mining over event streams, subgraph matching in trace graphs, and normalization of tool calls. The macro system operates as a reversible execution compression pipeline.

The process follows four steps:

1. **Propose** — a macro is proposed, assisted by LLM analysis of trace patterns
2. **Validate** — tested against historical execution data
3. **Compile** — once validated, compiled into a reusable execution unit
4. **Demote** — can be demoted if it falls out of use or if underlying primitives change

Macros are compiled execution subgraphs, not behavior overrides. They are an optimization layer for execution efficiency and reliability. They never replace primitives, and they remain fully reversible.

## Macros as Compiled Commit Ranges

Frequently occurring subgraphs are identified, validated, and compiled. A macro can always be expanded back into its originating events. This guarantees that no behavior is ever hidden behind macro abstraction — the full provenance chain remains accessible.

## Promotion Criteria

Not every repeated pattern should become a macro. A pattern is promoted only when it meets multiple criteria:

- **Frequency** — the pattern occurs often enough that compilation yields measurable efficiency gains
- **Success rate** — the pattern completes successfully across diverse contexts, not just in narrow conditions
- **Stability** — the underlying primitives and data shapes are unlikely to change in ways that would invalidate the macro
- **Cost savings** — the macro reduces compute time, token usage, or resource consumption compared to raw execution
- **Semantic equivalence** — the macro's behavior is functionally identical to the original pattern, not an approximation

These criteria prevent the system from generating mountains of garbage macros — low-frequency, brittle, or marginally useful patterns that add complexity without value.

## Discovery as an Open Problem

Detecting useful reusable patterns is fundamentally difficult. The macro system's discovery mechanism is an active area of development. The criteria above provide a starting framework, but the actual mining algorithms, subgraph matching strategies, and semantic equivalence detection remain open research questions. The system is designed to evolve its discovery mechanism without changing the macro compilation, validation, or execution model.
