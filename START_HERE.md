# Start Here

> **B.Y.T.E. — Behavior Yielding Through Evolution**

---

## What Is This?

B.Y.T.E. is a system that learns from its own experience. Instead of re-thinking every problem from scratch, it builds up a library of what it has learned — facts it knows, patterns it has seen, and solutions it has found — so that over time, it needs less and less computation to do the same work.

Think of it like a person who gets better at their job not by getting smarter, but by building better habits and reference materials.

**The core question this project tests:** Can accumulated structure substitute for repeated inference?

In plain terms: Can a system get better by _organizing what it has learned_ rather than by _thinking harder_?

---

## Choose Your Path

### I want to understand what this is

**Start here →** [CONCEPTUAL_OVERVIEW.md](docs/CONCEPTUAL_OVERVIEW.md)

- What problem this solves
- How it differs from ChatGPT, Siri, and existing AI assistants
- What the user experience is like
- What the system knows about you and how it protects that
- Reading time: ~5 minutes

### I want to understand how it works

**Start here →** [README.md](README.md) → [TECHNICAL_CONCEPT.md](docs/TECHNICAL_CONCEPT.md#3-architecture-overview)

- Full architecture specification
- Core invariants and design philosophy
- All subsystems and their interactions
- Reading time: ~30 minutes

### I want to evaluate the research

**Start here →** [docs/core/EVALUATION.md](docs/core/EVALUATION.md#primary-metric-reasoning-cost-per-task) → [docs/core/KNOWN_GAPS.md](docs/core/KNOWN_GAPS.md#macro-discovery-section-91)

- Falsifiable thesis and evaluation methodology
- External baseline comparisons
- Ablation study design
- Known gaps and future work
- Reading time: ~15 minutes

---

## System at a Glance

```
   ┌─────────────┐
   │     You     │
   └──────┬──────┘
          │   voice, gaze, gestures
          ▼
   ┌─────────────┐
   │  Edge Node  │   ← wearable device (glasses, earbuds, etc.)
   │   (fast)    │     processes sensors locally, runs immediate responses
   └──────┬──────┘
          │   structured events only (no raw audio/video)
          ▼
   ┌─────────────┐
   │   Homelab   │   ← your home server
   │   (deep)    │     long-term memory, heavy reasoning, planning
   └──────┬──────┘
          │   normalized requests only
          ▼
   ┌─────────────┐
   │  AI Models  │   ← cloud or local LLMs
   │  (optional) │     used as reasoning coprocessors, not the control system
   └─────────────┘
```

**Key principle:** Your raw data (audio, video, biometrics) never leaves your edge device. Only structured, compressed events flow to the homelab. AI models receive only normalized requests — they never control the system.

---

## A Day in the Life

**Morning.** You wake up. The system already knows your schedule because it learned your patterns over weeks. It offers a brief briefing: today's weather, your first meeting, a reminder about the package arriving. You didn't ask — it offered, because it learned that you like morning briefings. You accept with a nod. Whether the system started with a morning briefing skill or discovered the pattern on its own, it has compiled a personalized macro that runs faster and matches your actual preferences.

**Commute.** Your edge device processes your surroundings in real time. It identifies your usual coffee shop, notes the queue length, and suggests an alternative route. All of this happens locally on the device — no data leaves your hardware.

**Work.** You ask the system to research a topic. It generates a plan, shows it to you, then executes. You can check progress at any time. When it finds relevant information from a past project, it pulls from memory rather than searching from scratch.

**Evening.** The system summarizes the day into a narrative memory: what happened, what was learned, what patterns emerged. Over time, these memories become validated knowledge — facts the system can rely on without re-deriving.

**Over weeks and months.** The system builds macros — compiled patterns of behavior that used to require full reasoning but now execute deterministically. Skills you may have installed are gradually compressed into personalized macros that carry provenance links back to their source. The weather check that once took 2,000 tokens now takes 200. The morning briefing that required planning now fires automatically from a learned schedule. When a skill is updated, the system automatically re-validates its derived macros.

---

## Document Map

| Document                                               | Audience  | Purpose                                          |
| ------------------------------------------------------ | --------- | ------------------------------------------------ |
| [CONCEPTUAL_OVERVIEW.md](docs/CONCEPTUAL_OVERVIEW.md)  | Everyone  | Plain-language explanation of the system         |
| [README.md](README.md)                                 | Everyone  | Project abstract, thesis, and navigation         |
| [docs/TECHNICAL_CONCEPT.md](docs/TECHNICAL_CONCEPT.md) | Technical | Full architecture specification (Sections 1–18)  |
| [docs/REFERENCE.md](docs/REFERENCE.md)                 | Everyone  | Glossary of all terms used in the documentation  |
| [docs/core/](docs/core/)                               | Technical | Detailed specifications of individual subsystems |

**Recommended reading order for first-time visitors:**

1. This document
2. [CONCEPTUAL_OVERVIEW.md](docs/CONCEPTUAL_OVERVIEW.md)
3. [README.md](README.md)
4. [docs/REFERENCE.md](docs/REFERENCE.md) (keep open as reference)
5. [docs/TECHNICAL_CONCEPT.md](docs/TECHNICAL_CONCEPT.md) (if you want technical depth)

---

## Current Status

This project is in the **design phase**. Implementation has not yet begun.

| Phase | Component                           | Status          |
| ----- | ----------------------------------- | --------------- |
| 1     | Kernel + Execution Graph            | Design complete |
| 2     | RPU + Orchestration                 | Design complete |
| 3     | Signal-to-Intent Pipeline           | Design complete |
| 4     | Memory + Knowledge + Skill Registry | Design complete |
| 5     | Macros + Code Registry              | Research phase  |
| 6     | Edge + Multimodal                   | Conceptual      |

Phases 1–4 test the core thesis. Phases 5–6 are experimental extensions.
