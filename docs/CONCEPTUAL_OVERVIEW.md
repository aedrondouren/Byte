# Conceptual Overview

> A plain-language explanation of B.Y.T.E. — what it does, why it matters, and how it differs from what exists today.

---

## The Problem

Current AI assistants — ChatGPT, Siri, Google Assistant, and the growing family of "AI agents" — share a fundamental flaw: **they forget everything between conversations.**

Every time you interact with them, they start from scratch. They don't remember what you told them last week. They don't learn from their mistakes. They don't build up knowledge about your preferences, your habits, or your world. To compensate, they use bigger models, longer context windows, and more complex prompts — all of which cost more money, take more time, and still don't solve the core problem.

It's like hiring an assistant who has amnesia every morning. No matter how smart they are, they'll always be inefficient.

There's a second problem: **current AI agents embed everything into a single conversation.** Memory, planning, personality, world knowledge, tool use, and communication are all crammed into one prompt. This makes agents expensive (every interaction requires full AI computation), fragile (no persistent state between sessions), and opaque (no audit trail of decisions).

---

## The Idea

B.Y.T.E. takes a different approach. Instead of making the AI smarter, it makes the **system** smarter by building structure around the AI.

The core idea is simple: **Can a system get better by organizing what it has learned, rather than by thinking harder?**

Here's how it works:

1. **(Optional) The user may install skills** — pre-authored behavior definitions in the same format used by existing AI harnesses. These give the system competent behavior from day one, but the system works perfectly without them.
2. **The system does things** — executes skills through reasoning (if installed), answers questions, runs automations, helps the user work.
3. **Everything that happens is recorded** — not as a conversation log, but as structured events: what was perceived, what was decided, what was done, what the outcome was.
4. **Experience is compressed into knowledge** — repeated patterns become facts the system knows. Sequences of events become memories. Frequently-used workflows become compiled routines (macros) that carry provenance links back to their source skills when applicable.
5. **Future tasks need less thinking** — because the system already knows the facts, remembers the context, and has pre-built routines for common patterns.

The mechanism is a continuous loop:

```
(Optional: Install skills) → Do something → Record what happened → Learn from it → Build structure → Need less thinking next time
```

This is the opposite of how most AI systems work. Instead of "bigger model = better," the approach is "better organization = less model needed." Skills give you optional competence at day one; macros give you optimized, personalized competence over time — with or without skills.

---

## How It Works (High Level)

### The World-State Graph

Everything the system knows is stored in a structure similar to how Git stores code history. Every perception, decision, and action is recorded as an immutable event. The current state of the world is derived from this history, not stored separately.

This means:

- **Nothing is lost.** The full history is always available.
- **Everything is traceable.** You can always answer: why did the system do X? What information did it have? What alternatives existed?
- **Nothing is hidden.** If the system makes a mistake, you can trace it back to its origin.

### The Kernel and the Coprocessor

The system has two main parts:

**The Kernel** is the permanent, deterministic core. It manages execution, scheduling, memory, and safety. It never depends on AI reasoning. It is the part that cannot be replaced without rebuilding the entire system.

**The RPU (Reasoning Processing Unit)** is a replaceable AI coprocessor. The kernel sends it structured requests — "here is the current situation, here is what needs to be done, here is relevant context" — and receives structured results — "here is the plan, here are the proposed actions, here is my confidence." The AI never controls the system; it only proposes. The kernel validates, schedules, and executes.

This separation means:

- **AI models can be swapped** without changing the system.
- **The system is safe** — the AI cannot execute arbitrary actions.
- **The system is efficient** — the AI is only called when genuine reasoning is needed.

### The Seven Knowledge Domains

The system organizes what it knows into seven domains, stored across two data models:

| Domain            | What It Stores                   | Example                                             | Store          |
| ----------------- | -------------------------------- | --------------------------------------------------- | -------------- |
| **Perception**    | What the system has observed     | "User is at their desk, working on a laptop"        | Event store    |
| **Execution**     | What the system has done         | "Fetched weather forecast, sent to user"            | Event store    |
| **Memory**        | What the system has experienced  | "User asked for weather every morning this week"    | Event store    |
| **Knowledge**     | What the system knows to be true | "User prefers morning weather briefings for Boston" | Event store    |
| **Skills**        | Pre-authored behavior templates  | Installed skill: morning briefing (from community)  | Artifact store |
| **Macros**        | Patterns that repeat             | Compiled routine: morning briefing workflow         | Artifact store |
| **Code Registry** | Tested code components           | Validated function: format weather data             | Artifact store |

Four domains (perception, execution, memory, knowledge) are derived from the system's lived experience as an append-only event stream. Three domains (macros, skills, code registry) are versioned artifacts — entities the system creates, manages, and evolves over time.

Skills are an optional seed — behaviors the user may install to accelerate early competence. The system works without skills, discovering patterns organically. As skills execute (when installed), the system learns from the execution traces and compiles personalized, optimized versions (macros) that carry provenance links back to the original skill.

### The Learning Pipeline

The system learns through a multi-stage pipeline:

1. **Perception** — raw sensor data becomes structured observations (object detection, speech-to-text, etc.)
2. **Situation Model** — observations are combined into understanding ("user is in a focused work session")
3. **Intent** — understanding becomes action ("offer assistance")
4. **Execution** — actions are carried out
5. **Memory** — outcomes are summarized into narrative memories
6. **Knowledge** — repeated patterns are validated into facts
7. **Macros** — frequently-used workflows are compiled into efficient routines

Each stage compresses the previous one. Raw sensor data (gigabytes per hour) becomes structured perception (kilobytes per hour), which becomes situation models (bytes per event), which becomes memories (one sentence per experience), which becomes knowledge (one fact per pattern).

---

## What You Experience

### As a User

You interact with the system through natural means — voice, gestures, gaze, or text. It responds contextually, drawing on everything it has learned about you and your environment.

**Key differences from current AI assistants:**

- **It remembers.** Not just the last conversation — everything, organized and compressed into useful knowledge.
- **It anticipates.** It learns your patterns and offers help proactively (with your approval).
- **It improves over time.** The more you use it, the less it needs to "think" to give you good answers.
- **It is transparent.** You can always ask: why did you do that? What do you know? How confident are you?
- **It works offline.** The edge device handles immediate responses locally. The homelab handles deep reasoning.

### As a Developer

The system provides a deterministic runtime for building AI-powered applications. The AI is a coprocessor with structured contracts, not the control flow. This means:

- **Predictable behavior.** The kernel executes deterministically; the AI only proposes.
- **Auditability.** Every action is traceable to its origin.
- **Model independence.** Swap AI models without changing application logic.
- **Reusable components.** A code registry provides tested, versioned building blocks.

---

## What It Knows About You

The system stores information about your preferences, habits, environment, and interactions. Here is how it handles that data:

**Raw data never leaves your device.** Audio, video, biometric signals (EEG, EMG, gaze tracking) are processed locally on the edge device. Only structured, compressed events flow to the homelab.

**You can inspect everything.** The full event history is accessible. You can see what the system knows, how it knows it, and how confident it is.

**Knowledge has expiration dates.** Facts carry validity windows. "User prefers Rust" may be true for a project phase but not indefinitely. When knowledge is not corroborated over time, its confidence decays.

**You can correct it.** When the system makes an incorrect inference, your correction is recorded as evidence that feeds back into the knowledge validation pipeline.

**AI models don't see your raw data.** Cloud AI providers receive only normalized inference requests — structured context and task definitions, not your personal information.

---

## How It's Different

|                  | ChatGPT / LLM Chatbots                    | AI Agent Frameworks                       | B.Y.T.E.                                            |
| ---------------- | ----------------------------------------- | ----------------------------------------- | --------------------------------------------------- |
| **Memory**       | Limited to conversation context           | External memory added as afterthought     | Built-in, event-sourced, compressed over time       |
| **Control**      | The model controls the conversation       | The model orchestrates tool calls         | A deterministic kernel controls; the model proposes |
| **Learning**     | No learning between sessions              | No learning between sessions              | Continuous learning through experience compression  |
| **Transparency** | Opaque reasoning                          | Partial audit trails                      | Full provenance chain for every action              |
| **Cost**         | Every interaction requires full inference | Every interaction requires full inference | Cost decreases over time as structure accumulates   |
| **Reliability**  | Model-dependent                           | Model-dependent                           | Kernel is permanent; model is replaceable           |
| **Privacy**      | Data sent to provider                     | Data sent to provider                     | Raw data stays on your device                       |

---

## What's Real vs. What's Planned

This project is in the **design phase**. Nothing has been implemented yet. Here is the current status:

| Phase | What It Adds                             | Status                                 |
| ----- | ---------------------------------------- | -------------------------------------- |
| **1** | Core kernel and execution system         | Design complete                        |
| **2** | AI reasoning coprocessor + orchestration | Design complete                        |
| **3** | Sensor-to-action pipeline                | Design complete                        |
| **4** | Memory, knowledge, and skill registry    | Design complete                        |
| **5** | Macros + code registry                   | Research phase — optimization layer    |
| **6** | Wearable edge device + multimodal input  | Conceptual — hardware not yet designed |

Phases 1–4 are the core system. Phase 4 (memory + knowledge) is the falsification point for the core thesis. Skills are available from Phase 4 as a harness completeness feature but are not part of the thesis test. Phase 5 adds the optimization layer: compiling skill execution traces and general patterns into deterministic macros. Phases 5–6 are experimental extensions built on that foundation.

---

## Next Steps

- **To explore the technical architecture:** [TECHNICAL_CONCEPT.md](TECHNICAL_CONCEPT.md#3-architecture-overview)
- **To understand the research evaluation:** [core/EVALUATION.md](core/EVALUATION.md#primary-metric-reasoning-cost-per-task)
- **To see what problems remain unsolved:** [core/KNOWN_GAPS.md](core/KNOWN_GAPS.md#macro-discovery-section-91)
- **To look up any term:** [REFERENCE.md](REFERENCE.md)
