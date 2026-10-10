# MATH-500

**TL;DR:** 500 competition math problems. Local best: Mirai S at 94.2%. The qwen3.8-27B family clusters at 85-90%; anything under 80% is a red flag for reasoning.

**What it evaluates:** 500 competition-style math problems (MATH dataset's hardest split), free-form answer, graded for exact correctness. Measures raw reasoning: multi-step algebra, combinatorics, probability, number theory. No tools, no code execution — pure chain-of-thought quality.

**Reading scores:** 85-87% is the 27B-class norm on this hardware. 89%+ is standout. 98% from a frontier remote model shows the gap between local and cloud reasoning. Sub-80% means the model can't sustain multi-step chains reliably.

## Results

| Model | Accuracy | Notes |
|---|---|---|
| nous-deepseek-v4-flash (remote) | 98.2% | remote baseline |
| **Mirai S Qwen3.8-27B** (2.4 bpw trellis) | **94.2%** | **best local** — see [model page](../models/mirai-s-qwen3-8-27b.md) |
| **OrcaSAQ-2-27B** (vLLM) | **90.8%** | 2nd local — math leader, 262K ctx |
| swift-qwen3.8-27b | 89.2% | best local (previous) |
| **Coder390-EfficientThink** (Q3LynnStyle-Q8MTP) | 89.0% | thinking-off run |
| qwen3.8-27b | 86.8% | |
| qwen3.8-27b-heretic | 85.8% | |
| exl3-qwen3.8-27b | 85.8% | same weights, different serving engine |
| bonsai2 (PrismML tern PTQ1_0) | 85.6% | 5.9 GB file, hybrid SSM |
| Signal-3.8-27B-AP | 85.6% | |
| GSQ-RCO-IQ3_S-mtp | 85.4% | 3.5bpw mixed-precision, ties larger quants |
| ornith-1.5-35b-a3b (q4_k_s) | 85.2% | 35B MoE, 21 GB, 64k ctx |
| qwen3-coder-30b | 85.0% | "coder" model, mid math |
| crack2 (abliterated PQ2_0) | 83.6% | 7.2 GB, weight-edit |
| **Muse Glimmer 30B** (Q4_K_M) | 82.6% | |
| **Nemotron Cascade 2 30B A3B** (Q3_K_M) | 81.2% | |
| hermes-4.3-36b | 81.0% | |
| Qwen3.8-27B-TurboFCFusion ("turbo-fable", Q4_K_M) | 81.0% | |
| gpt-oss-20b-mxfp4 | 74.2% | dead last |
| Holo4-27B (Q4_K_M, Q8 KV) | 73.4% | |
| Spark-X2.5-4B (Q4_K_M) | 72.2% | strong for 4B |
| gemma-4-26b | 48.8% | rejected — far below field |

**Scoring:** `math_verify` — extracts answer from `\boxed{...}` in the model's response and compares against the gold answer. Partial credit: none. 500 individual items, no multiple choice.

**Hardware:** RTX 3090 24 GB, llama.cpp GGUF server, quantized models from HuggingFace.