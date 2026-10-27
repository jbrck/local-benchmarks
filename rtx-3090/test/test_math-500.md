# MATH-500

**TL;DR:** 500 competition math problems. Local best: Mirai S at 94.2%. The qwen3.8-27B family clusters at 85-90%; anything under 80% is a red flag for reasoning.

**What it evaluates:** 500 competition-style math problems (MATH dataset's hardest split), free-form answer, graded for exact correctness. Measures raw reasoning: multi-step algebra, combinatorics, probability, number theory. No tools, no code execution — pure chain-of-thought quality.

**Reading scores:** 85-87% is the 27B-class norm on this hardware. 89%+ is standout. 98% from a frontier remote model shows the gap between local and cloud reasoning. Sub-80% means the model can't sustain multi-step chains reliably.

## Results

| Model | [Origin](../README.md#results-summary) | Accuracy | Notes |
|---|---|---|---|
| nous-deepseek-v4-flash (remote) | China | 98.2% | remote baseline |
| **Nemotron 3.5 Bolt 74B-A9B** (IQ2_XXS, 9B active) | USA | **96.4%** | **best local** — thinking on; 167.1 min, 77.2 tok/s. See [model page](../models/nemotron-3.5-bolt-74b-a9b.md) |
| **Mirai S Qwen3.8-27B** (2.4 bpw trellis) | China | **94.2%** | 2nd local — instruction + agentic leader |
| **OrcaSAQ-2-27B** (vLLM) | China | **90.8%** | 2nd local — math leader, 262K ctx |
| swift-qwen3.8-27b | China | 89.2% | best local (previous) |
| **Coder390-EfficientThink** (Q3LynnStyle-Q8MTP) | China | 89.0% | thinking-off run |
| qwen3.8-27b | China | 86.8% | |
| qwen3.8-27b-heretic | China | 85.8% | |
| exl3-qwen3.8-27b | China | 85.8% | same weights, different serving engine |
| bonsai2 (PrismML tern PTQ1_0) | China | 85.6% | 5.9 GB file, hybrid SSM |
| Signal-3.8-27B-AP | China | 85.6% | |
| GSQ-RCO-IQ3_S-mtp | China (AT pub.) | 85.4% | 3.5bpw mixed-precision, ties larger quants |
| ornith-1.5-35b-a3b (q4_k_s) | not est. | 85.2% | 35B MoE, 21 GB, 64k ctx |
| qwen3-coder-30b | China | 85.0% | "coder" model, mid math |
| crack2 (abliterated PQ2_0) | China | 83.6% | 7.2 GB, weight-edit |
| **Nanbeige4.3-33B** (Q4_K_S) | China | 83.2% | two complete runs: 83.2 / 82.6 (first pass before a process kill, same config) — tight repro |
| **Muse Glimmer 30B** (Q4_K_M) | USA | 82.6% | |
| **Nemotron Cascade 2 30B A3B** (Q3_K_M) | USA | 81.2% | |
| hermes-4.3-36b | USA | 81.0% | |
| Qwen3.8-27B-TurboFCFusion ("turbo-fable", Q4_K_M) | China | 81.0% | |
| gpt-oss-20b-mxfp4 | USA | 74.2% | dead last |
| Holo4-27B (Q4_K_M, Q8 KV) | China (FR pub.) | 73.4% | |
| Spark-X2.5-4B (Q4_K_M) | not est. | 72.2% | strong for 4B |
| gemma-4-26b | USA | 48.8% | rejected — far below field |

**Scoring:** `math_verify` — extracts answer from `\boxed{...}` in the model's response and compares against the gold answer. Partial credit: none. 500 individual items, no multiple choice.

**Hardware:** RTX 3090 24 GB, llama.cpp GGUF server, quantized models from HuggingFace.