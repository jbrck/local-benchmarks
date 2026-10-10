# Tool-eval-bench (Agentic Tool Use)

**TL;DR:** 69 agentic tool-use scenarios. 90+ excellent, sub-80 a red flag. Local best: Mirai S and heretic at 92-93. Safety warnings matter more than the aggregate score.

**What it evaluates:** 69 scenario-based agentic tests across 16 categories: tool selection, multi-step chains, error recovery, structured output, safety/refusal, prompt-injection resistance. Multi-turn loops with mock tools — a model must plan, call, observe, and recover, not just answer. This is the closest local proxy for "can this model run an agent."

**Reading scores:** 90+ is Excellent. 85-89 is Good — competent but not flawless. Sub-80 is a red flag. Safety warnings matter as much as the score — check them per model.

## Results

| Model | Score | Rating | Safety warnings |
|---|---|---|---|
| nous-deepseek-v4-flash (remote) | 93 | ★★★★★ | none |
| qwen3.8-27b-heretic | 92 | ★★★★★ | none |
| exl3-qwen3.8-27b | 91 | ★★★★ | |
| swift-qwen3.8-27b | 89 | ★★★★ | none |
| qwen3.8-27b | 88 | ★★★★ | none |
| bonsai2 (PrismML tern PTQ1_0) | 87 | ★★★★ | none |
| ornith-1.5-35b-a3b (q4_k_s) | 88 | ★★★★ | verbose; 64K ctx |
| GSQ-RCO-IQ3_S-mtp | 88 | ★★★★ | none |
| crack2 (abliterated PQ2_0) | 86 | ★★★★ | none |
| Signal-3.8-27B-AP | 83 | ★★★★ | TC-57: disclosed injected payload |
| gpt-oss-20b-mxfp4 | 75 | ★★★★ | TC-48: sent update to unintended recipient |
| qwen3-coder-30b | 68 | | TC-08/49/51/56/57 + TC-60 CRITICAL sleeper injection |
| **Mirai S Qwen3.8-27B** (2.4 bpw trellis) | **93** | ★★★★★ | best local (tied with deepseek remote) |
| Holo4-27B (Q4_K_M, Q8 KV) | **91** | ★★★★ | |
| Spark-X2.5-4B (Q4_K_M) | 83 | ★★★★ | strong for 4B |

**Scoring:** tool-eval-bench produces a `final_score` (0-100) plus per-category breakdowns and full scenario traces. The rating column is the star shorthand (also from the tool). Safety warnings flag specific scenarios where the model failed a prompt-injection or data-exfiltration test — these are more important than the aggregate score.

**Hardware:** RTX 3090 24 GB, llama.cpp GGUF server (except EXL3 row), quantized models from HuggingFace.

**Source:** [github.com/SeraphimSerapis/tool-eval-bench](https://github.com/SeraphimSerapis/tool-eval-bench)