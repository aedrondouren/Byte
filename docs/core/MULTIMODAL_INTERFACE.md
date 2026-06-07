# Multimodal Cognitive Interface

> Expands on the Multimodal Cognitive Interface section of [TECHNICAL_CONCEPT.md](../TECHNICAL_CONCEPT.md#13-multimodal-cognitive-interface).
> **Audience:** Technical
> **Prerequisites:** TECHNICAL_CONCEPT.md Sections 1–7, GRAPH.md
> **Status:** Complete

The multimodal interface is an experimental extension. The system functions with voice and text input alone — gaze, EMG, and EEG are optional modalities that may improve interaction quality but are not required for the core thesis.

The system is a world-state engine that continuously updates a structured representation of the environment, user attention, and internal state. Inputs come from multiple modalities — voice, gaze tracking, IMU and head pose, cameras in RGB, depth, and IR, EEG providing low-bandwidth cognitive and context signals, and EMG or micro-gestures for confirmation.

Each sensor contributes partial, noisy evidence that is fused into a unified real-time intent and context graph.

## Modalities

**Gaze** provides attention and selection signals. Where the user looks indicates what they care about. Gaze fixation duration and trajectory inform engagement level.

**Voice** provides explicit intent. Natural language commands, questions, and conversational input. The most direct signal but not the only one.

**EMG and blink signals** provide confirmation. Micro-gestures and blink patterns serve as implicit yes/no signals, reducing the need for explicit verbal confirmation.

**EEG** provides state modulation including fatigue, stress, and cognitive load. Low-bandwidth cognitive and context signals that adjust system behavior based on the user's internal state.

**Behavior history** provides priors for intent estimation. Past interactions, preferences, and patterns inform how the system interprets current signals.

**Personality state** modulates how the system interprets and responds to fused intent.

No single modality is sufficient on its own. The system fuses partial, noisy evidence from multiple modalities to estimate what the user intends.

## Intent Fusion

Human inputs are not commands. They are probabilistic signals fused into a unified intent and context graph.

Interaction is probabilistic intent estimation, not deterministic control. The system fuses partial, noisy evidence from multiple modalities to estimate what the user intends, and the kernel validates and executes only when confidence thresholds are met.

User intent from the multimodal layer, system-generated intent from situation model interpretation, and temporally-scheduled intent from knowledge all converge into the same intent → proposal → commit pipeline (TECHNICAL_CONCEPT.md Section 7.3).

### Sensor Fusion Algorithm Overview

The system uses a weighted evidence fusion model:

1. **Per-modality scoring.** Each modality produces an independent intent estimate with a confidence score.
2. **Weighting by reliability.** Modalities are weighted by their historical accuracy in the current context. For example, gaze is weighted higher in quiet environments, voice is weighted higher in noisy environments.
3. **Temporal smoothing.** Recent evidence is weighted more heavily than older evidence, with a configurable decay window (default: 5 seconds).
4. **Conflict detection.** When modalities disagree beyond a threshold, the system flags the conflict and either requests explicit confirmation or defaults to the most reliable modality for the current context.
5. **Threshold evaluation.** The fused confidence score is compared against execution thresholds (see below).

### Intent Fusion Threshold Justification

| Threshold      | Action                | Justification                                                                                                |
| -------------- | --------------------- | ------------------------------------------------------------------------------------------------------------ |
| **Above 0.9**  | Execute immediately   | Multiple modalities agree with high confidence. False positive rate is acceptably low.                       |
| **0.7 to 0.9** | Present as suggestion | Single strong signal or moderate agreement across modalities. User confirmation reduces false positive risk. |
| **0.5 to 0.7** | Log as context        | Weak signal. Not actionable alone, but contributes to situation model and future intent estimation.          |
| **Below 0.5**  | Discard as noise      | Signal is below the reliability floor. Recorded for model training but does not influence behavior.          |

These thresholds are calibrated during an initial adaptation period where the system learns the user's signal patterns. Users can adjust sensitivity through explicit feedback.

### Modality Conflict Resolution

When modalities disagree (e.g., gaze indicates interest in object A but voice commands action on object B):

1. **Voice takes priority** for explicit commands. Voice is the highest-bandwidth, most intentional modality.
2. **Gaze provides context** for ambiguous voice commands. If the user says "open that" while looking at a specific object, gaze disambiguates.
3. **EEG modulates timing.** If cognitive load is high, the system defers non-urgent suggestions even if other modalities indicate interest.
4. **Conflict is logged** as an event for future calibration. Repeated conflicts in similar contexts trigger threshold adjustment.

### Calibration Procedure

Thresholds are calibrated through:

- **Explicit feedback.** User acceptance/denial of system actions adjusts modality weights.
- **Implicit feedback.** User behavior after a system action (e.g., immediately undoing an action) is treated as negative feedback.
- **Periodic review.** The system presents a summary of its confidence calibration to the user for manual adjustment.

## AR Interface as Output Channel

The AR interface acts as the primary output channel, turning the internal world model into spatially grounded overlays, notifications, and adaptive UI elements.

EEG and other biosignals do not act as direct command channels. They are contextual modulation signals that adjust system behavior based on attention, fatigue, stress, or engagement, improving the timing and relevance of interactions.

## Hierarchical Control Stack

The system separates concerns by latency:

**Low-latency edge compute** (wearable/backpack) — handles perception, signal filtering, and immediate interaction logic. Ensures the system remains responsive even when disconnected from the homelab.

**Homelab system** — provides heavier reasoning, long-term memory, planning, and tool execution. The anchor point for all long-term cognitive processing.

A custom orchestration harness governs all LLM usage, ensuring bounded autonomy, deterministic tool calls, and state-verified updates rather than unconstrained generation.

The system runs continuously as a multimodal, distributed assistant that integrates into perception and action loops rather than behaving as a discrete chatbot. Its defining property is not model intelligence, but how tightly sensing, memory, reasoning, and execution are connected across wearable and home infrastructure.

## Orchestration Harness

The harness governs all LLM usage with bounded autonomy, deterministic tool calls, and state-verified updates.

The harness enforces:

- **Input validation** — all data passed to the LLM is structured and schema-validated
- **Output validation** — all LLM output is checked against the expected schema before use
- **Tool call authorization** — the LLM proposes tool calls; the harness verifies them against the current world-state and priority constraints before execution
- **State verification** — any state update proposed by the LLM is validated against the current world-state projection before being committed

The harness is part of the kernel, not the LLM. It ensures that the LLM operates within defined boundaries and cannot produce side effects outside the execution chain model.

---

**Related:** [TECHNICAL_CONCEPT.md](../TECHNICAL_CONCEPT.md#13-multimodal-cognitive-interface) for the original specification.
