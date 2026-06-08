# Document Map

> Document dependency graph, reading order recommendations, and status tracking for the B.Y.T.E. documentation set.
> **Audience:** Everyone
> **Status:** Complete

---

## Document Dependency Graph

```
START_HERE.md (entry point)
├──→ CONCEPTUAL_OVERVIEW.md (plain-language)
│   └──→ REFERENCE.md (glossary)
└──→ README.md (project overview)
    ├──→ TECHNICAL_CONCEPT.md (core specification, Sections 1–18)
    │   ├──→ core/GRAPH.md (world-state graph)
    │   ├──→ core/RPU.md (reasoning coprocessor)
    │   ├──→ core/ORCHESTRATION.md (execution primitives)
    │   ├──→ core/MACROS.md (macro system)
    │   ├──→ core/REGISTRY.md (code registry)
    │   ├──→ core/EDGE_ARCHITECTURE.md (edge node)
    │   ├──→ core/MULTIMODAL_INTERFACE.md (multimodal input)
    │   ├──→ core/OFFLINE_OPTIMIZATION.md (background loop)
    │   ├──→ core/SECURITY.md (security & privacy)
    │   ├──→ core/EVALUATION.md (evaluation methodology)
    │   ├──→ core/KNOWN_GAPS.md (open problems)
    │   ├──→ core/DESIGN_DECISIONS.md (rationale)
    │   ├──→ core/THREAT_MODEL.md (threat analysis)
    │   └──→ core/ARCHITECTURE_DIAGRAMS.md (flow diagrams)
    └──→ REFERENCE.md (glossary)
```

---

## Reading Order by Audience

### Non-Technical Readers

1. [START_HERE.md](../START_HERE.md) — orientation and system overview
2. [CONCEPTUAL_OVERVIEW.md](CONCEPTUAL_OVERVIEW.md) — plain-language explanation
3. [REFERENCE.md](REFERENCE.md) — keep open as reference for terms

### Technical Readers (First Pass)

1. [START_HERE.md](../START_HERE.md) — orientation
2. [README.md](../README.md) — project abstract and navigation
3. [REFERENCE.md](REFERENCE.md) — terminology (keep open)
4. [TECHNICAL_CONCEPT.md](TECHNICAL_CONCEPT.md#1-introduction) — core specification (Sections 1–18)
5. [core/DESIGN_DECISIONS.md](core/DESIGN_DECISIONS.md) — rationale behind choices

### Technical Readers (Deep Dive)

After first pass, read core documents as needed:

- [core/GRAPH.md](core/GRAPH.md) — world-state graph, query complexity
- [core/RPU.md](core/RPU.md) — RPU contract, error handling, model versioning
- [core/ORCHESTRATION.md](core/ORCHESTRATION.md) — primitive semantics, composition laws
- [core/MACROS.md](core/MACROS.md) — discovery, validation, demotion
- [core/REGISTRY.md](core/REGISTRY.md) — versioning, dependency resolution, transpilation

### Researchers

1. [TECHNICAL_CONCEPT.md](TECHNICAL_CONCEPT.md#1-introduction) Sections 1–4 (thesis and foundation)
2. [core/EVALUATION.md](core/EVALUATION.md#primary-metric-reasoning-cost-per-task) — methodology, baselines, ablation
3. [core/KNOWN_GAPS.md](core/KNOWN_GAPS.md#macro-discovery-section-91) — open problems
4. [core/DESIGN_DECISIONS.md](core/DESIGN_DECISIONS.md) — rationale

### Security Reviewers

1. [core/SECURITY.md](core/SECURITY.md) — security specification
2. [core/THREAT_MODEL.md](core/THREAT_MODEL.md#assumed-attacker-capabilities) — threat scenarios and mitigations
3. [TECHNICAL_CONCEPT.md](TECHNICAL_CONCEPT.md#14-core-invariants) (invariants), [TECHNICAL_CONCEPT.md](TECHNICAL_CONCEPT.md#15-security-and-privacy-considerations) (security)

---

## Document Status

| Document                                                       | Status   | Audience             |
| -------------------------------------------------------------- | -------- | -------------------- |
| [START_HERE.md](../START_HERE.md)                              | Complete | Everyone             |
| [README.md](../README.md)                                      | Complete | Everyone             |
| [CONCEPTUAL_OVERVIEW.md](CONCEPTUAL_OVERVIEW.md)               | Complete | Everyone             |
| [REFERENCE.md](REFERENCE.md)                                   | Complete | Everyone             |
| [DOCUMENT_MAP.md](DOCUMENT_MAP.md)                             | Complete | Everyone             |
| [TECHNICAL_CONCEPT.md](TECHNICAL_CONCEPT.md#1-introduction)    | Complete | Technical            |
| [core/GRAPH.md](core/GRAPH.md)                                 | Complete | Technical            |
| [core/RPU.md](core/RPU.md)                                     | Complete | Technical            |
| [core/ORCHESTRATION.md](core/ORCHESTRATION.md)                 | Complete | Technical            |
| [core/MACROS.md](core/MACROS.md)                               | Complete | Technical            |
| [core/REGISTRY.md](core/REGISTRY.md)                           | Complete | Technical            |
| [core/EDGE_ARCHITECTURE.md](core/EDGE_ARCHITECTURE.md)         | Complete | Technical            |
| [core/MULTIMODAL_INTERFACE.md](core/MULTIMODAL_INTERFACE.md)   | Complete | Technical            |
| [core/OFFLINE_OPTIMIZATION.md](core/OFFLINE_OPTIMIZATION.md)   | Complete | Technical            |
| [core/SECURITY.md](core/SECURITY.md)                           | Complete | Technical            |
| [core/EVALUATION.md](core/EVALUATION.md)                       | Complete | Technical / Research |
| [core/KNOWN_GAPS.md](core/KNOWN_GAPS.md)                       | Complete | Technical / Research |
| [core/DESIGN_DECISIONS.md](core/DESIGN_DECISIONS.md)           | Complete | Technical            |
| [core/THREAT_MODEL.md](core/THREAT_MODEL.md)                   | Complete | Technical / Security |
| [core/ARCHITECTURE_DIAGRAMS.md](core/ARCHITECTURE_DIAGRAMS.md) | Complete | Technical            |

---

## Cross-Reference Index

| Referenced By                 | References                                                                                                                                                                                                                                                                                                          |
| ----------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| TECHNICAL_CONCEPT.md          | core/GRAPH.md, core/RPU.md, core/ORCHESTRATION.md, core/MACROS.md, core/REGISTRY.md, core/EDGE_ARCHITECTURE.md, core/MULTIMODAL_INTERFACE.md, core/OFFLINE_OPTIMIZATION.md, core/SECURITY.md, core/EVALUATION.md, core/KNOWN_GAPS.md, core/DESIGN_DECISIONS.md, core/THREAT_MODEL.md, core/ARCHITECTURE_DIAGRAMS.md |
| README.md                     | START_HERE.md, CONCEPTUAL_OVERVIEW.md, REFERENCE.md, TECHNICAL_CONCEPT.md, all core documents                                                                                                                                                                                                                       |
| START_HERE.md                 | CONCEPTUAL_OVERVIEW.md, README.md, REFERENCE.md, TECHNICAL_CONCEPT.md                                                                                                                                                                                                                                               |
| CONCEPTUAL_OVERVIEW.md        | TECHNICAL_CONCEPT.md, core/EVALUATION.md, core/KNOWN_GAPS.md, REFERENCE.md                                                                                                                                                                                                                                          |
| core/RPU.md                   | TECHNICAL_CONCEPT.md, core/ORCHESTRATION.md, core/MACROS.md                                                                                                                                                                                                                                                         |
| core/ORCHESTRATION.md         | TECHNICAL_CONCEPT.md, core/MACROS.md, core/REGISTRY.md                                                                                                                                                                                                                                                              |
| core/MACROS.md                | TECHNICAL_CONCEPT.md, core/ORCHESTRATION.md, core/RPU.md, core/SECURITY.md (skill-derived validation)                                                                                                                                                                                                               |
| core/REGISTRY.md              | TECHNICAL_CONCEPT.md, core/ORCHESTRATION.md                                                                                                                                                                                                                                                                         |
| core/EDGE_ARCHITECTURE.md     | TECHNICAL_CONCEPT.md                                                                                                                                                                                                                                                                                                |
| core/MULTIMODAL_INTERFACE.md  | TECHNICAL_CONCEPT.md                                                                                                                                                                                                                                                                                                |
| core/SECURITY.md              | TECHNICAL_CONCEPT.md, core/THREAT_MODEL.md                                                                                                                                                                                                                                                                          |
| core/THREAT_MODEL.md          | core/SECURITY.md, TECHNICAL_CONCEPT.md                                                                                                                                                                                                                                                                              |
| core/EVALUATION.md            | TECHNICAL_CONCEPT.md, core/KNOWN_GAPS.md                                                                                                                                                                                                                                                                            |
| core/KNOWN_GAPS.md            | TECHNICAL_CONCEPT.md, core/EVALUATION.md, core/DESIGN_DECISIONS.md                                                                                                                                                                                                                                                  |
| core/DESIGN_DECISIONS.md      | TECHNICAL_CONCEPT.md, core/KNOWN_GAPS.md                                                                                                                                                                                                                                                                            |
| core/GRAPH.md                 | TECHNICAL_CONCEPT.md                                                                                                                                                                                                                                                                                                |
| core/OFFLINE_OPTIMIZATION.md  | TECHNICAL_CONCEPT.md                                                                                                                                                                                                                                                                                                |
| core/ARCHITECTURE_DIAGRAMS.md | TECHNICAL_CONCEPT.md, core/GRAPH.md                                                                                                                                                                                                                                                                                 |
| REFERENCE.md                  | All documents (terminology reference)                                                                                                                                                                                                                                                                               |
