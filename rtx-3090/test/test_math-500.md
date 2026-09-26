# MATH-500

**What it evaluates:** 500 competition-style math problems (MATH dataset's hardest split), free-form answer, graded for exact correctness. Measures raw reasoning: multi-step algebra, combinatorics, probability, number theory. No tools, no code execution — pure chain-of-thought quality.

**Reading scores:** 85-87% is the 27B-class norm on this hardware. 89%+ is standout. 98% from a frontier remote model shows the gap between local and cloud reasoning. Sub-80% means the model can't sustain multi-step chains reliably.

## Results

| Model | Accuracy | Notes |
|---|---|---|
| nous-deepseek-v4-flash (remote) | 98.2% | remote baseline |
| **OrcaSAQ-2-27B** (vLLM) | **90.8%** | **best local** — hybrid attention, 262K ctx |
| swift-qwen3.8-27b | 89.2% | best local (previous) |
| qwen3.8-27b | 86.8% | |
| bonsai2 (PrismML tern PTQ1_0) | 85.6% | 5.9 GB file, hybrid SSM |
| orcarouter (runtime LoRA 2.0) | 85.4% | 5.9 GB + 9 MB LoRA |
| ornith-1.5-35b-a3b (q4_k_s) | 85.2% | 35B MoE, 21 GB, 64k ctx |
| qwen3.8-27b-heretic | 85.8% | |
| crack2 (abliterated PQ2_0) | 83.6% | 7.2 GB, weight-edit |
| exl3-qwen3.8-27b | 85.8% | same weights, different serving engine |
| Signal-3.8-27B-AP | 85.6% | |
| GSQ-RCO-IQ3_S-mtp | 85.4% | 3.5bpw mixed-precision, ties larger quants |
| qwen3-coder-30b | 85.0% | "coder" model, mid math |
| hermes-4.3-36b | 81.0% | |
| gpt-oss-20b-mxfp4 | 74.2% | dead last |

**Scoring:** `math_verify` — extracts answer from `\boxed{...}` in the model's response and compares against the gold answer. Partial credit: none. 500 individual items, no multiple choice.

**Hardware:** RTX 3090 24 GB, llama.cpp GGUF server, quantized models from HuggingFace.