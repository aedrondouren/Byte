> This document expands on the Multimodal Cognitive Interface section of [TECHNICAL_CONCEPT.md](../TECHNICAL_CONCEPT.md).

You are designing a **distributed multimodal personal cognitive interface system**: a layered architecture that connects your homelab AI to a wearable AR/edge node through continuous sensor fusion, persistent memory, and tight orchestration loops.

At its core, the system is not a single “assistant,” but a **world-state engine** that continuously updates a structured representation of your environment, your attention, and your internal state. Inputs come from multiple modalities—voice, gaze tracking, IMU/head pose, cameras (RGB/depth/IR), EEG (low-bandwidth cognitive/context signals), and EMG or micro-gestures for confirmation. Each sensor contributes partial, noisy evidence that is fused into a unified real-time “intent + context graph.”

This fused state is processed through a **hierarchical control stack**: low-latency wearable or backpack compute handles perception, signal filtering, and immediate interaction logic, while a homelab system provides heavier reasoning, long-term memory, planning, and tool execution. A custom orchestration harness governs all LLM usage, ensuring bounded autonomy, deterministic tool calls, and state-verified updates rather than unconstrained generation.

The AR interface acts as the primary output layer, turning this internal world model into spatially grounded overlays, notifications, and adaptive UI elements. EEG and other biosignals do not act as direct command channels, but as **contextual modulation signals**—adjusting system behavior based on attention, fatigue, stress, or engagement, improving timing and relevance of interactions.

Overall, the system is a **persistent ambient AI layer**: a continuously running, multimodal, distributed assistant that integrates into perception and action loops rather than behaving as a discrete chatbot. Its defining property is not model intelligence, but the tightness of its feedback loop between sensing, memory, reasoning, and execution across wearable and home infrastructure.
