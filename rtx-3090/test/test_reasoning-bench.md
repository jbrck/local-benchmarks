# Reasoning Bench (Thinking Tokens & Speed)

**TL;DR:** 8 tasks measuring thinking-token efficiency. Mirai S reasons 1.41x faster than vanilla qwen with 21.7% fewer thinking tokens. Spark's 0/8 is a format artifact, not capability.

**What it evaluates:** 8 tasks with reasoning ENABLED, measuring completion tokens, estimated thinking tokens, wall time, and correctness. The tool for testing "N% fewer thinking tokens / Nx speedup" claims on community merge cards. Always compare against the qwen3.8-27b-reasoning baseline on this exact hardware.

**Reading scores:** Better = fewer thinking tokens AND faster wall time at equal accuracy. swift delivers: 21.7% fewer thinking tokens, 1.41x wall speedup (its card claimed 58.3%/1.95x — direction true, magnitude overstated). Twin-Turbo's card claimed 1/2-to-1/20 thinking tokens; it measured +44% MORE thinking and 0.59x wall time — the claim was inverted. Marketing numbers on merge cards are unreliable in both directions.

## Results

| Model | Correct | Thinking tokens (est) | Wall time | vs baseline |
|---|---|---|---|---|
| qwen3.8-27b (baseline) | 8/8 | 614 | 29.9s | 1.0x |
| swift-qwen3.8-27b | 8/8 | 481 (-21.7%) | 21.2s | **1.41x** |
| Twin-Turbo | 8/8 | 885 (+44%) | 50.5s | 0.59x |

**Correctness axis note:** All models score 8/8 on this specific 8-task set — the correctness axis is saturated and cannot rank models. The useful signal is thinking-token count and wall time.

**Scoring:** Thinking tokens are estimated from the `reasoning_content` field character count divided by 4 (approximate token ratio). Wall time is end-to-end per task including API latency. The baseline is qwen3.8-27b with reasoning enabled on the same server, same GPU.

**Hardware:** RTX 3090 24 GB, llama.cpp GGUF server with reasoning mode enabled.