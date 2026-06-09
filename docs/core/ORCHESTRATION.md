# Semantic Orchestration Layer

> Expands on the Semantic Orchestration Layer section of [TECHNICAL_CONCEPT.md](../TECHNICAL_CONCEPT.md#10-semantic-orchestration-layer).
> **Audience:** Technical
> **Prerequisites:** TECHNICAL_CONCEPT.md Sections 1–5, 10
> **Status:** Complete

## How vs What

The semantic orchestration layer defines **how** things run. The derived graphs define **what** is available: macros for runtime execution, code components for software development.

- **Macros** consume Tool Services through orchestration primitives
- **Code components** consume Application Services through orchestration primitives

The two ecosystems never intersect.

## Core Primitives — Semantic Specifications

These primitives are modeled on Effect.ts semantics. Each primitive is language-agnostic — the semantics are portable, the implementation is language-specific.

### `run(task)`

Execute a task and return its result.

- **Semantics:** Sequential execution. Blocks until the task completes or fails.
- **Failure:** Propagates the task's error to the enclosing scope.
- **Composition:** `run(a) |> run(b)` executes `a` then `b`. If `a` fails, `b` is not executed.

### `fork(task)`

Spawn a concurrent task that runs independently.

- **Semantics:** The forked task runs concurrently with the parent. The parent continues without waiting.
- **Failure:** If the forked task fails, the error is captured and can be observed via `join`. Unobserved fork failures are logged but do not propagate to the parent.
- **Lifecycle:** Forked tasks are tracked by the scheduler and can be cancelled via `scope`.
- **Composition:** `fork(a) |> fork(b)` spawns both `a` and `b` concurrently.

### `parallel(taskA, taskB, ...)`

Run multiple tasks concurrently and wait for all to complete.

- **Semantics:** All tasks start concurrently. The primitive returns when all tasks complete.
- **Failure:** If any task fails, all other tasks are cancelled and the first error is propagated.
- **Composition:** `parallel(a, b)` is commutative — `parallel(a, b)` is equivalent to `parallel(b, a)`.
- **Associativity:** `parallel(a, parallel(b, c))` is equivalent to `parallel(a, b, c)`.

### `race(taskA, taskB, ...)`

Run multiple tasks concurrently and take the first result.

- **Semantics:** All tasks start concurrently. The primitive returns with the result of the first task to complete.
- **Failure:** If all tasks fail, the last error is propagated. If one task succeeds before others fail, the success is returned.
- **Cancellation:** When the first task completes, all other tasks are cancelled.
- **Composition:** `race(a, b)` is commutative.

### `scope(config)`

Manage task lifecycle and cancellation.

- **Semantics:** Establishes a scope within which tasks execute. The scope can be configured with timeout, cancellation signals, and resource limits.
- **Cancellation:** When the scope is cancelled (via timeout, explicit signal, or parent cancellation), all tasks within the scope are cancelled.
- **Cleanup:** Scope guarantees that cleanup handlers run even if tasks are cancelled.
- **Composition:** Scopes can be nested. Cancelling a parent scope cancels all child scopes.

### `acquire(resource) / release(resource)`

Resource management with guaranteed cleanup.

- **Semantics:** `acquire` obtains a resource. `release` is guaranteed to run after the scoped block completes, regardless of success or failure.
- **Failure:** If `acquire` fails, the scoped block does not execute and `release` is not called. If the scoped block fails, `release` still runs.
- **Composition:** `acquire(a) |> acquire(b)` acquires `a` then `b`. On release, `b` is released before `a` (LIFO order).
- **Guarantee:** `release` runs even if the task is cancelled via `scope`.

### `fail(error) / recover(handler) / mapError(transform)`

Error handling and transformation.

- **`fail(error)`:** Explicitly produces a failed effect with the given error.
- **`recover(handler)`:** Catches an error and runs a recovery handler. If the handler succeeds, its result is returned. If the handler fails, the new error propagates.
- **`mapError(transform)`:** Transforms the error value without changing success/failure status.
- **Composition:** `task |> recover(handler) |> mapError(transform)` — recovery runs first, then error transformation applies to any remaining errors.

### `stream(source) / pipe(transform) / sink(destination)`

Streaming data flow.

- **`stream(source)`:** Creates a stream from a data source. Emits values over time.
- **`pipe(transform)`:** Applies a transformation to each stream value. Transforms are composable: `pipe(a) |> pipe(b)` applies `a` then `b` to each value.
- **`sink(destination)`:** Consumes the stream and delivers values to a destination. Runs until the stream ends or the scope is cancelled.
- **Failure:** If the source fails, the stream terminates with an error. If a transform fails, the stream terminates with the transform's error.
- **Backpressure:** Streams support backpressure — if the sink is slower than the source, the source is throttled.

### `provide(layer) / layer(service) / context(key)`

Dependency injection and service composition.

- **`provide(layer)`:** Provides a service layer to the enclosed effects. The layer satisfies the service requirements of effects within the scope.
- **`layer(service)`:** Creates a service layer from a service implementation. Layers can be composed: `layer(a) |> layer(b)` provides both services.
- **`context(key)`:** Accesses a service from the current context. Fails if the service is not provided.
- **Composition:** Layers are composable and can override previous layers. The innermost layer takes precedence.

## Composition Examples

A macro that checks weather and sends a notification:

```
acquire(weatherService)
  |> run(fetchForecast(location))
  |> mapError(logAndRecover)
  |> parallel(
       run(sendNotification(user, forecast)),
       run(updateExecutionGraph(forecast))
     )
  |> scope(cancelOnTimeout(30s))
  |> release
```

The primitives compose deterministically. The macro can be expanded back into its constituent tool calls and scheduler decisions at any time.

A code component that processes user data:

```
provide(userDatabaseLayer)
  |> acquire(userService)
  |> run(validateInput(data))
  |> fork(run(persistToDatabase(data)))
  |> fail(notifyOnFailure(user))
  |> recover(retryWithBackoff(3))
  |> release
```

The same primitives, different services.

## Composition Laws

### Commutativity

| Primitive                      | Commutative? | Notes                                                    |
| ------------------------------ | ------------ | -------------------------------------------------------- |
| `parallel(a, b)`               | Yes          | Order does not affect result                             |
| `race(a, b)`                   | Yes          | Order does not affect result (both start simultaneously) |
| `run(a)` then `run(b)`         | No           | Sequential order matters                                 |
| `acquire(a)` then `acquire(b)` | No           | LIFO release order depends on acquisition order          |

### Associativity

| Primitive                     | Associative? | Notes                               |
| ----------------------------- | ------------ | ----------------------------------- |
| `parallel(a, parallel(b, c))` | Yes          | Equivalent to `parallel(a, b, c)`   |
| `pipe(a)` then `pipe(b)`      | Yes          | Function composition is associative |
| `layer(a)` then `layer(b)`    | Yes          | Layer composition is associative    |

### Identity

| Primitive                       | Identity Element                    | Notes                      |
| ------------------------------- | ----------------------------------- | -------------------------- |
| `parallel(a, unit)`             | `unit` (no-op effect)               | Equivalent to `run(a)`     |
| `pipe(a)` then `pipe(identity)` | `identity` (pass-through transform) | No effect on stream values |

## Error Propagation Rules

Errors propagate through composition according to these rules:

1. **Sequential composition (`run`):** Errors propagate forward. If `a` fails, `b` is not executed.
2. **Parallel composition (`parallel`):** The first error cancels all other tasks and propagates.
3. **Race composition (`race`):** Errors are ignored if another task succeeds. If all tasks fail, the last error propagates.
4. **Fork composition (`fork`):** Errors are captured but do not propagate unless explicitly joined.
5. **Recovery (`recover`):** Errors are caught and handled. The recovery handler's result replaces the error.
6. **Scope cancellation:** All in-progress tasks are cancelled. Cleanup handlers run. No error propagates from cancellation itself.

## Resource Lifecycle Guarantees

The orchestration layer provides these guarantees:

- **Acquired resources are always released.** Even if the task fails, is cancelled, or the process crashes (within the limits of the runtime's crash recovery).
- **Forked tasks are tracked.** The scheduler knows about all active forks and can cancel them via scope.
- **Parallel tasks are atomically cancelled.** If one task in a parallel group fails, all others are cancelled before the error propagates.
- **Stream sinks are closed.** When a stream ends (normally or via error), the sink's cleanup runs.

## Reference: Effect.ts Semantics

This orchestration layer is modeled on [Effect.ts](https://effect.website/). The primitives map to Effect's core types:

| Byte Primitive          | Effect Equivalent                                     |
| ----------------------- | ----------------------------------------------------- |
| `run(task)`             | `Effect.flatMap`                                      |
| `fork(task)`            | `Effect.fork`                                         |
| `parallel(...)`         | `Effect.all([...], { concurrency: 'unbounded' })`     |
| `race(...)`             | `Effect.raceAll`                                      |
| `scope(config)`         | `Effect.scoped`                                       |
| `acquire/release`       | `Effect.acquireRelease`                               |
| `fail/recover/mapError` | `Effect.fail` / `Effect.catchAll` / `Effect.mapError` |
| `stream/pipe/sink`      | `Stream` module                                       |
| `provide/layer/context` | `Layer` / `Context` modules                           |

The semantic equivalence ensures that TypeScript + Effect code can be used as the reference implementation, with transpilation to other runtimes preserving semantics.

---

**Related:** [TECHNICAL_CONCEPT.md](../TECHNICAL_CONCEPT.md#10-semantic-orchestration-layer) for the original specification. [MACROS.md](MACROS.md#what-macros-are) for how macros consume Tool Services through these primitives. [REGISTRY.md](REGISTRY.md#code-registry) for how code components consume Application Services through these primitives.
