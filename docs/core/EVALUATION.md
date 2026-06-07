# Evaluation Methodology

> Expands on the Evaluation section of [TECHNICAL_CONCEPT.md](../TECHNICAL_CONCEPT.md#16-evaluation-methodology).
> **Audience:** Technical / Research
> **Prerequisites:** TECHNICAL_CONCEPT.md Sections 1–4
> **Status:** Complete

The core thesis — that accumulated structure can substitute for repeated inference — is falsifiable. This section defines how it will be tested.

## Primary Metric: Reasoning Cost per Task

The primary metric is the **reasoning cost** required to achieve a given outcome, measured in:

- **Token consumption** — total tokens used per task (input + output)
- **Inference latency** — wall-clock time from intent to completion
- **RPU invocations** — number of model calls required

These are tracked per task type and aggregated over time.

## Baseline

The baseline is established in Phase 2 (RPU + Orchestration): reasoning cost for tasks using planning-first workflows with structured contracts but no memory, knowledge, or macros. This represents the "structured but not compressed" baseline.

## Measurement Points

| Phase   | What Is Added                        | Expected Effect                                                                                            |
| ------- | ------------------------------------ | ---------------------------------------------------------------------------------------------------------- |
| Phase 2 | Planning-first, structured contracts | Minimal reduction: plan reuse avoids re-derivation                                                         |
| Phase 3 | Signal-to-intent pipeline            | Moderate reduction: structured context replaces ad-hoc prompting                                           |
| Phase 4 | Memory + Knowledge graphs            | **Significant reduction**: validated facts eliminate re-derivation, narrative memories replace raw context |
| Phase 5 | Macros                               | Additional reduction: compiled execution patterns replace reasoning entirely for repeated tasks            |

## Falsification Criterion

If Phase 4 does not demonstrate a **statistically significant reduction** (p < 0.05) in reasoning cost for equivalent task outcomes compared to the Phase 2 baseline, the core thesis fails. "Equivalent outcomes" means the same task completed to the same quality standard.

## Control Variables

To ensure valid measurement:

- The same model family is used across all phases (model version is controlled)
- Task complexity is held constant within each measurement cohort
- The same evaluation harness measures cost across all phases
- Results are aggregated over a minimum of 100 task executions per task type

## External Baseline Comparison

In addition to internal phase-to-phase comparison, Byte will be evaluated against existing agent frameworks on identical tasks:

| Framework                        | Comparison Dimension                                                          |
| -------------------------------- | ----------------------------------------------------------------------------- |
| **LangGraph**                    | Token consumption, task success rate, reasoning cost over repeated executions |
| **AutoGen**                      | Multi-agent coordination cost, memory persistence, execution auditability     |
| **CrewAI**                       | Role-based task completion cost, learning across sessions                     |
| **Custom prompt-chain baseline** | Direct comparison to a well-engineered prompt-chain agent on the same tasks   |

The comparison uses the same task taxonomy, the same model family where possible, and the same quality standard for "equivalent outcomes." This ensures that Byte's improvement is measured not just against itself but against the state of the art.

## Ablation Study Design

To isolate the contribution of each subsystem, the following ablation studies will be conducted:

| Ablation                 | What Is Removed                               | What Is Measured                                     |
| ------------------------ | --------------------------------------------- | ---------------------------------------------------- |
| **No knowledge graph**   | Knowledge validation pipeline disabled        | Impact of validated facts vs. raw memory alone       |
| **No memory graph**      | Narrative memory and retrieval disabled       | Impact of experience recall vs. no recall            |
| **No macros**            | Macro discovery and compilation disabled      | Impact of execution compression vs. raw execution    |
| **No temporal intent**   | Temporal pattern extraction disabled          | Impact of proactive automation vs. reactive only     |
| **No personality state** | Personality projection replaced with defaults | Impact of personality adaptation vs. static behavior |

Each ablation is run for 100 task executions per task type, with results compared to the full system and the Phase 2 baseline.

## Statistical Power Analysis

The minimum of 100 task executions per task type is based on a power analysis targeting:

- **Effect size:** Cohen's d = 0.5 (medium effect)
- **Alpha:** 0.05 (standard significance threshold)
- **Power:** 0.80 (80% chance of detecting a true effect)
- **Test:** Two-sample t-test (Phase 2 baseline vs. Phase 4)

This sample size provides sufficient power to detect a 20% reduction in reasoning cost with 95% confidence.

## Task Taxonomy

Tasks are categorized by complexity and domain:

| Category                  | Examples                                   | Expected Compression Benefit                         |
| ------------------------- | ------------------------------------------ | ---------------------------------------------------- |
| **Information retrieval** | Weather check, calendar lookup, fact query | High — facts become knowledge, queries become macros |
| **Workflow execution**    | Multi-step task with tool calls            | High — repeated workflows become macros              |
| **Creative generation**   | Writing, brainstorming, design             | Low — inherently novel, less compressible            |
| **Problem solving**       | Debugging, analysis, planning              | Medium — patterns emerge but context varies          |
| **Conversation**          | Open-ended dialogue                        | Low — highly contextual, less compressible           |

The taxonomy ensures that evaluation covers both compressible and non-compressible tasks, preventing the thesis from being validated only on easy cases.

## Quality Standard Definition

"Equivalent outcomes" is defined per task type:

- **Information retrieval:** Correct answer with the same accuracy and completeness.
- **Workflow execution:** Same end state achieved (e.g., same files created, same API calls made).
- **Creative generation:** Subjective quality rated by blinded human evaluators on a 1–5 scale; scores within 0.5 points are considered equivalent.
- **Problem solving:** Same solution correctness; alternative valid solutions are accepted.
- **Conversation:** Task completion (e.g., information conveyed, decision reached) rather than conversational quality.

## Secondary Metrics

- **Knowledge graph growth rate** — how quickly validated facts accumulate
- **Macro hit rate** — percentage of executions served by macros vs. full reasoning
- **Temporal intent accuracy** — percentage of temporally-scheduled intents accepted vs. denied by the user
- **Confidence calibration** — how well confidence scores predict actual correctness
- **Degradation behavior** — system performance when components are removed (simulating failures)

---

**Related:** [TECHNICAL_CONCEPT.md](../TECHNICAL_CONCEPT.md#16-evaluation-methodology) for the original specification. [KNOWN_GAPS.md](KNOWN_GAPS.md) for evaluation limitations.
