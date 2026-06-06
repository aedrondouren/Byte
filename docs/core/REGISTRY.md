# Code Registry

> This document expands on the Code Registry section (Section 8.2) of [TECHNICAL_CONCEPT.md](../TECHNICAL_CONCEPT.md).

## What Code Components Are

Code components are Effect-style function definitions that compose Application Services through semantic orchestration primitives. They use Application Services only — they never use Tool Services.

Code components support transpilation across language runtimes:

- **TypeScript + Effect → Go** — nearly 1:1 transpilation due to structural similarity in concurrency models (goroutines, contexts, channels map closely to Effect primitives)
- **TypeScript + Effect → C++** — requires a runtime layer to support the same semantic primitives, reserved for environments like Unreal Engine that expect native code

Native code modules can be exposed as Application Services, enabling cross-language composition.

## The Problem

Most application code consists of common patterns: data transformation, validation, formatting, state management, error handling. These are repeatedly written, rarely tested thoroughly, and pulled from external packages that may carry security risks or unnecessary dependencies.

## The Solution

Build an npm-like repository where:

- Code components are written once, tested thoroughly, and versioned
- The model pulls from the registry when writing new code instead of reinventing logic
- Every component has a full history — who wrote it, what tests it passes, what depends on it, how it evolved
- Quality improves through reuse: frequently used components get more testing, edge cases get discovered, they get refined
- Safety comes from versioning: you know exactly what code is running, it's been validated, and you can roll back

## How It Works

Code components are actively written — by the model during task execution, by autonomous agents working on specific pieces, or by the user directly. Every new version must pass a mandatory test pipeline before adoption:

- Full test suite execution
- Coverage thresholds maintained or improved
- No regressions in dependent components
- Compatibility with declared dependencies

Only components that pass all checks are published. Failed versions are recorded in the event log but never become available for use.

## External Dependencies

Core packages remain external: frameworks, bundlers, compilers, database drivers. These are infrastructure you don't reimplement.

But the logic that glues them together becomes internal: parsers, validators, formatters, state machines, utility functions, domain-specific logic. Over time, the system accumulates a tested, versioned library of in-house components that replaces external dependencies for small logic.

## Edge Node Access

The registry lives on the homelab as the trusted source. Edge nodes pull from it when needed — initially rarely, but as the system grows, edge devices including small laptop setups and the PEN may pull tested components for local execution. Components are pinned to specific versions and verified through cryptographic hashes.
