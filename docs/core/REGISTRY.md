# Code Registry

> Expands on the Code Registry section of [TECHNICAL_CONCEPT.md](../TECHNICAL_CONCEPT.md#92-code-registry).
> **Audience:** Technical
> **Prerequisites:** TECHNICAL_CONCEPT.md Section 9.2, ORCHESTRATION.md
> **Status:** Complete

The code registry is a repository of tested, versioned code components stored in the artifact version store, used when writing software — not executed by the runtime. When the model, agents, or users build applications together, they pull from the registry instead of generating code from scratch.

Code component definitions live in the artifact version store. Test runs and adoption events are recorded in the world-state event store.

## Dependency Taxonomy

**Stays external** — infrastructure you don't reimplement:

- Frameworks (React, Effect, etc.)
- Bundlers (esbuild, webpack)
- Compilers (TypeScript, Go toolchain)
- Database drivers (pg, sqlite)
- Protocol implementations (HTTP, WebSocket, gRPC)

**Becomes internal** — logic that glues infrastructure together:

- Parsers and serializers for domain-specific formats
- Validators for business rules
- Formatters for output generation
- State machines for workflow logic
- Utility functions for data transformation
- Domain-specific logic (user preferences, project configuration, notification routing)

## Transpilation Details

### TypeScript + Effect → Go

Nearly 1:1 transpilation due to structural similarity:

| Effect primitive  | Go equivalent                          |
| ----------------- | -------------------------------------- |
| `Effect<T, E>`    | `func() (T, error)`                    |
| `fork`            | goroutine with context                 |
| `scope`           | `defer` + context cancellation         |
| `acquire/release` | `defer` pattern                        |
| `provide/layer`   | dependency injection via struct fields |
| `stream`          | channels                               |
| `race`            | `select` statement                     |
| `parallel`        | `errgroup` or `sync.WaitGroup`         |

Goroutines, contexts, and channels map closely to Effect primitives, making the transpilation straightforward for most patterns.

### TypeScript + Effect → C++

Requires a runtime layer to support the same semantic primitives. Reserved for environments like Unreal Engine that expect native code. The runtime layer provides:

- Fiber-based concurrency (since C++ lacks native async/await in the same model)
- RAII-based resource management for acquire/release
- Exception-based error handling mapped to fail/recover
- Custom stream implementations for streaming data flow

Native code modules can be exposed as Application Services, enabling cross-language composition.

## Versioning Mechanics

### Content-Addressed Versioning

Every code component version is identified by its cryptographic hash (SHA-256). This ensures:

- **Immutability:** Once published, a component version cannot be changed.
- **Reproducibility:** The same hash always produces the same code.
- **Integrity verification:** Edge nodes can verify component integrity before use.

### Semantic Version Tags

In addition to content-addressed hashes, components carry semantic version tags (`major.minor.patch`) for human readability:

- **Major:** Breaking API changes. Dependents must update their code.
- **Minor:** New features, backward-compatible. Dependents can update without changes.
- **Patch:** Bug fixes, backward-compatible. Dependents should update.

Semantic versions are mutable (a tag can be moved to a new hash), but the hash is the canonical identifier. The registry always resolves tags to hashes.

### Version Pinning and Edge Access

The registry lives on the homelab as the trusted source. Edge nodes pull from it when needed. Components are pinned to specific versions and verified through cryptographic hashes before use.

Version pinning works as follows:

1. An edge node declares its required component versions in a manifest.
2. The homelab verifies that all requested versions exist and pass their test pipeline.
3. Components are transferred with their cryptographic hashes.
4. The edge node verifies hashes before making components available for code generation.
5. If a component fails verification, the edge node falls back to a known-good version or requests a fresh copy.

No untested or unversioned code is ever distributed. Failed component versions are recorded in the event log but never become available for use.

## Dependency Resolution

### Dependency Graph Traversal

When a component is requested, the registry resolves its full dependency tree:

1. **Direct dependencies** are resolved from the component's manifest.
2. **Transitive dependencies** are resolved recursively.
3. **Version conflicts** are detected when two components require incompatible versions of the same dependency.
4. **Resolution strategy:** The registry attempts to find a version that satisfies all constraints. If no such version exists, the conflict is reported and the request fails.

### Conflict Resolution

When version conflicts occur:

1. **Compatible range intersection:** If the required version ranges overlap, the highest version in the intersection is selected.
2. **No intersection:** The conflict is reported with the conflicting components and their version requirements. The user or model must resolve the conflict manually.
3. **Circular dependencies:** Detected during graph traversal and reported as an error. Circular dependencies are not permitted.

## Component Deprecation Policy

Components can be deprecated but never deleted:

- **Deprecation** marks a component version as not recommended for new use. Existing dependents continue to function.
- **Deprecation reason** is recorded (security vulnerability, superseded by newer component, known bug).
- **Replacement suggestion** is provided when available.
- **Deprecated components** are excluded from search results and auto-suggestions but remain accessible for existing dependents.

## Registry Query Interface

The model and agents query the registry through a structured interface:

```typescript
interface RegistryQuery {
	capability: string; // What the component does (e.g., "format date", "validate email")
	language?: string; // Target language (TS, Go, C++)
	constraints?: {
		// Version, dependency, or performance constraints
		minVersion?: string;
		maxDependencies?: number;
		maxComplexity?: string;
	};
}

interface RegistryResult {
	component: ComponentRef; // Hash, name, version, description
	matchScore: number; // How well the component matches the query
	dependencies: ComponentRef[];
	testResults: TestSummary; // Pass rate, coverage, last run
}
```

The query interface uses semantic search (capability description → component match) combined with structured filters (language, constraints). The model receives ranked results and selects the best match.

---

**Related:** [TECHNICAL_CONCEPT.md](../TECHNICAL_CONCEPT.md#92-code-registry) for the original specification. [ORCHESTRATION.md](ORCHESTRATION.md#core-primitives--semantic-specifications) for the primitives code components consume. [TECHNICAL_CONCEPT.md](../TECHNICAL_CONCEPT.md#93-skill-registry) for the relationship between macros, code components, and skills.
