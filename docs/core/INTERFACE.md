# Multimodal Cognitive Interface

> This document expands on the Multimodal Cognitive Interface section (Section 11) of [TECHNICAL_CONCEPT.md](../TECHNICAL_CONCEPT.md).

## Overview

The system is a world-state engine that continuously updates a structured representation of the environment, user attention, and internal state. It is not a single "assistant" but a **persistent ambient AI layer** — a continuously running, multimodal, distributed system that integrates into perception and action loops rather than behaving as a discrete chatbot.

Its defining property is not model intelligence, but the tightness of its feedback loop between sensing, memory, reasoning, and execution across wearable and home infrastructure.

## Modalities

Inputs come from multiple modalities. Each sensor contributes partial, noisy evidence that is fused into a unified real-time intent and context graph.

**Gaze** — provides attention and selection signals. Where the user looks indicates what they care about. Gaze fixation duration and trajectory inform engagement level.

**Voice** — provides explicit intent. Natural language commands, questions, and conversational input. The most direct signal but not the only one.

**EMG and blink signals** — provide confirmation. Micro-gestures and blink patterns serve as implicit yes/no signals, reducing the need for explicit verbal confirmation.

**EEG** — provides state modulation including fatigue, stress, and cognitive load. Low-bandwidth cognitive and context signals that adjust system behavior based on the user's internal state.

**Behavior history** — provides priors for intent estimation. Past interactions, preferences, and patterns inform how the system interprets current signals.

No single modality is sufficient on its own. The system fuses partial, noisy evidence from multiple modalities to estimate what the user intends.

## Intent Fusion

Human inputs are not commands. They are probabilistic signals fused into a unified intent and context graph.

Interaction is **probabilistic intent estimation**, not deterministic control. The system fuses partial, noisy evidence from multiple modalities to estimate what the user intends, and the kernel validates and executes only when confidence thresholds are met.

User intent from the multimodal layer converges with system-generated intent from semantic interpretation. When the system detects that the user has been stuck on a problem for 30 minutes (semantic: focused work session + increasing cognitive load), it may propose offering help. Both user-originated and system-originated intent flow through the same three-stage pipeline:

1. **Intent** — state change request (what is desired)
2. **Proposal** — executable candidate (how to do it)
3. **Commit** — kernel-approved execution (authorization to proceed)

The kernel does not distinguish between user-originated and system-originated intent at the intent stage — both flow through the same validation, proposal generation, and commit pipeline.

## Hierarchical Control Stack

The system separates concerns by latency:

**Low-latency edge compute** (wearable/backpack) — handles perception, signal filtering, and immediate interaction logic. Ensures the system remains responsive even when disconnected from the homelab.

**Homelab system** — provides heavier reasoning, long-term memory, planning, and tool execution. The anchor point for all long-term cognitive processing.

A custom orchestration harness governs all LLM usage, ensuring bounded autonomy, deterministic tool calls, and state-verified updates rather than unconstrained generation.

## AR Interface as Output Channel

The AR interface acts as the **primary output channel**, turning the internal world model into spatially grounded overlays, notifications, and adaptive UI elements.

EEG and other biosignals do not act as direct command channels. They are **contextual modulation signals** — adjusting system behavior based on attention, fatigue, stress, or engagement, improving the timing and relevance of interactions.

The system is a continuously running, multimodal, distributed assistant that integrates into perception and action loops. It is not a chatbot. It is an ambient cognitive layer that extends the user's perception, memory, and reasoning into the physical world.
