# Semantic Orchestration Layer

> This document expands on the Semantic Orchestration Layer section (Section 7) of [TECHNICAL_CONCEPT.md](../TECHNICAL_CONCEPT.md).

## Overview

The system defines a minimal set of Effect-like orchestration primitives that are semantically equivalent across all runtime backends. These primitives define _what a program means_, independent of any programming language.

## Core Primitives

- `run` — execute a task
- `fork` — spawn a concurrent task
- `parallel` — run multiple tasks concurrently
- `race` — run multiple tasks, take the first result
- `scope` — manage task lifecycle and cancellation
- `acquire` / `release` — resource management with guaranteed cleanup
- `fail` / `recover` / `mapError` — error handling and transformation
- `stream` / `pipe` / `sink` — streaming data flow
- `provide` / `layer` / `context` — dependency injection and service composition

These are not a domain-specific language. They are a **portable execution meaning layer**.

## Service Ecosystems

Above the primitives sits a service-based composition layer. The system maintains two separate service ecosystems that never intersect:

### Tool Services

Exposed to macros. Individual tool calls are exposed as service methods. Macros compose these services through semantic primitives to execute automations. Tool Services are runtime-only — they are not transpilable.

### Application Services

Exposed to code components. Application-level capabilities — databases, APIs, storage, native code modules. Code components compose these through semantic primitives. Application Services support transpilation across language runtimes.

Code components use Application Services only. Macros use Tool Services only. The two ecosystems never mix.

## Relationship to Derived Graphs

The semantic orchestration layer defines _how_ things run. The derived graphs (Section 8 of TECHNICAL*CONCEPT.md) define \_what* is available to run:

- **Macros** consume Tool Services through orchestration primitives
- **Code components** consume Application Services through orchestration primitives
