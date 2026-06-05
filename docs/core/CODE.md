> This document expands on the Semantic Orchestration Layer and Code Registry sections of [TECHNICAL_CONCEPT.md](../TECHNICAL_CONCEPT.md).

## Semantic Orchestration Runtime

You're building a **semantic execution runtime for agents**, not a framework or DSL.

At the core is a small, stable set of **Effect-like orchestration primitives** (tasks, scopes, resources, concurrency, error handling, streams, and cancellation). These define _what a program means_, independent of any programming language.

Above that sits a **service-based composition layer**, where external capabilities (DBs, APIs, storage, AI tools, engine APIs, etc.) are expressed as injectable services with consistent contracts. The key rule is that services define the "leaves" of execution, while the core workflow logic remains pure and portable.

The system then provides multiple **runtime adapters**:

- A TypeScript runtime (likely Effect-based) for web/backend orchestration
- A Go runtime that maps the same semantics onto goroutines, contexts, and native concurrency primitives
- A future C++ runtime layer (e.g., Unreal Engine integration) using a wrapper runtime over engine tasking and lifecycle systems

Each adapter does not translate code literally; it **implements the same execution semantics using native primitives of the host environment**.

The result is a system where:

- The _workflow logic is portable and language-agnostic_
- The _runtime behavior is native and optimized per platform_
- The _differences between environments are isolated to service implementations and runtime adapters, not business logic_

In effect, it is a **cross-language semantic orchestration layer for agent execution**, where TypeScript, Go, and C++ are simply different execution backends for the same underlying computational model.

---

## Code Registry

The same semantic runtime pattern applies to the code the model writes. The code registry is a homelab-hosted, versioned repository where every code component — written by the model, by autonomous agents, or by the user — is tested, versioned, and stored as a reusable artifact.

### The Problem

Most application code consists of common patterns: data transformation, validation, formatting, state management, error handling. These are repeatedly written, rarely tested thoroughly, and pulled from external packages that may carry security risks or unnecessary dependencies.

### The Solution

Build an npm-like repository where:

- Code components are written once, tested thoroughly, and versioned
- The model pulls from the registry when writing new code instead of reinventing logic
- Every component has a full history — who wrote it, what tests it passes, what depends on it, how it evolved
- Quality improves through reuse: frequently used components get more testing, edge cases get discovered, they get refined
- Safety comes from versioning: you know exactly what code is running, it's been validated, and you can roll back

### How It Works

Code components are actively written — by the model during task execution, by autonomous agents working on specific pieces, or by the user directly. Every new version must pass a mandatory test pipeline before adoption:

- Full test suite execution
- Coverage thresholds maintained or improved
- No regressions in dependent components
- Compatibility with declared dependencies

Only components that pass all checks are published. Failed versions are recorded in the event log but never become available for use.

### Relationship to External Dependencies

Core packages remain external: frameworks, bundlers, compilers, database drivers. These are infrastructure you don't reimplement.

But the logic that glues them together becomes internal: parsers, validators, formatters, state machines, utility functions, domain-specific logic. Over time, the system accumulates a tested, versioned library of in-house components that replaces external dependencies for small logic.

### Relationship to Macros

Macros and code components are two applications of the same pattern — repeated behavior captured, validated, and made reusable. The distinction:

- Macros are passively discovered through trace mining, validated against historical execution data, and compress execution patterns
- Code components are actively written, must pass a mandatory test pipeline, and compress implementation patterns

Both follow the "history is more important than current state" principle. Both live in separate logical graphs to keep queries simple.

### Edge Node Access

The registry lives on the homelab as the trusted source. Edge nodes pull from it when needed — initially rarely, but as the system grows, edge devices including small laptop setups and the PEN may pull tested components for local execution. Components are pinned to specific versions and verified through cryptographic hashes.
