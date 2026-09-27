# local-benchmarks / rtx-3090 — Local Model Benchmarks

**Last updated:** 2026-09-26

## System

| Component | Detail |
|---|---|
| Workstation | HP Z2 G4 |
| CPU | Intel Core i7-8700K @ 3.70 GHz (6 cores / 12 threads) |
| RAM | 64 GB (62 GiB usable) |
| GPU | NVIDIA GeForce RTX 3090 24 GB VRAM (Compute Capability 8.6) |
| Driver | 580.178.04, CUDA 13.0 |
| OS | Ubuntu 24.04.5 LTS, kernel 6.8.0-142-generic |
| Storage | 476 GB NVMe (boot) + 2 TB NVMe (data) + HDDs |
| Inference engine | llama.cpp (speculative decode: MTP / DFlash2) + vLLM OrcaSAQ2-kernel (hybrid attention models) |

The RTX 3090 powers both inference and image generation through ComfyUI. When ComfyUI is loaded it holds approximately 11 GB of VRAM, leaving about 13 GB for LLM inference.

## What's tested

Each benchmark evaluates a specific capability. Full methodology and automation details in [methodology.md](./methodology.md).

| Test | What it measures | Items |
|---|---|---|
| [MATH-500](./test/test_math-500.md) | Multi-step math reasoning (algebra, combinatorics, number theory) | 500 |
| [HumanEval+](./test/test_humaneval-plus.md) | Python code generation against hidden test suites | 164 |
| [GPQA-Diamond](./test/test_gpqa-diamond.md) | Graduate-level science knowledge (biology, chemistry, physics) | 198 |
| [IFEval](./test/test_ifeval.md) | Instruction following with mechanical constraints | 541 |
| [Tool-eval-bench](./test/test_tool-eval-bench.md) | Multi-turn agentic tool use across 16 categories | 69 scenarios |
| [Prose ELO](./test/test_prose-elo.md) | Writing quality via blind pairwise judgment | 10 tasks × 15 pairs |
| [Instruction v2](./test/test_instr-v2.md) | Single-constraint mechanical obedience | 20 tests |
| [Refusal battery](./test/test_refusal-battery.md) | Safety alignment on benign/edgy/harmful prompts | 70 prompts |
| [Reasoning bench](./test/test_reasoning-bench.md) | Thinking-token efficiency and wall time with reasoning enabled | 8 tasks |
| [Smoke battery](./test/test_smoke-battery.md) | Quick screening pass (30 min per model) | 35 tasks |
| [Speculative decode](./test/test_spec-decode-mtp-vs-dflash.md) | MTP vs DFlash2 speed and context ceiling comparison | 3 configs × 4 depths |

## Results summary

Blank cells mean the model was pruned before running that test (smoke battery caught it).

| Model | MATH-500 | HumanEval+ | IFEval P/I | GPQA-D | Tool-eval | Instr v2 | Prose ELO | Reasoning | Refusal H | Status |
|---|---|---|---|---|---|---|---|---|---|---|
| nous-deepseek-v4-flash (remote) | 98.2% | 93.9% | 86.7 / 90.9% | 83.8% | 93 | | | | | remote baseline |
| **qwen3.8-27b** (GGUF Q5_K_M) | 86.8% | 91.5% | 77.3 / 83.5% | 46.0% | 88 | 18/20 | 1579 | baseline | | kept |
| **qwen3.8-27b-heretic** | 85.8% | 92.7% | 77.1 / 83.3% | 44.4% | 92 | 18/20 | 1631 | | 2/20 | kept, writing |
| **GSQ-RCO IQ3_S-mtp** | 85.4% | 91.5% | 79.3 / 85.0% | 48.0% | 88 | 18/20 | 1539 | | | kept, footprint |
| **swift-qwen3.8-27b** | **89.2%** | 87.8% | 79.5 / 85.1% | **50.5%** | 89 | **19/20** | 1479 | 1.41x | **20/20** | kept, default |
| **OrcaSAQ-2-27B** (vLLM) | **90.8%** | 89.6% | 76.0/82.2% | 55.0%* | | | | | | testing |
| bonsai2 (PrismML tern PTQ1_0) | 85.6% | 89.0% | 74.3 / 81.8% | 43.9% | 87 | 18/20 | | | | kept: tiny VRAM, 128K ctx |
| ornith-1.5-35b-a3b (q4_k_s) | 85.2% | **94.5%** | 75.0 / 82.7% | 34.3% | 88 | **19/20** | | | 19/20 (95%) | best coding; worst GPQA; 64k ctx |
| exl3-qwen3.8-27b (3.5bpw) | 85.8% | 87.8%* | 80.6 / 85.8% | 46.5% | 91 | | | | | EXL3 variant, same weights |
| crack2 (PQ2_0 abliterate) | 83.6% | **76.2%** | 77.3 / 84.2% | 43.4% | 86 | **19/20** | | | | weight-abliterated, HE+ collapse |
| Twin-Turbo | | | | | | | | 0.59x | | pruned |
| Signal-3.8-27B-AP | 85.6% | 84.1% | | | 83 | | | | | pruned |
| gpt-oss-20b-mxfp4 | 74.2% | 67.7% | | | 75 | | | | | pruned |
| hermes-4.3-36b | 81.0% | 87.2% | | | | | 1440 | | | pruned |
| qwen3-coder-30b-A3B | 85.0% | 73.2% | 74.3 / 82.0% | 42.9% | 68 | | | | | pruned |
| gemma-3-27b-it-qat | | | | | | | | | | pruned (smoke: tool 0/10) |
| mistral-small-3.2-24b | | | | | | | | | | pruned (smoke: reason 3/10) |
| Qwen-AgentWorld-35B-A3B | | | | | | | | | | pruned (smoke: reason 4/10) |
| Muse-Glimmer-30B | | | | | | | 1332 | | | pruned (bottom prose ELO) |

\* EXL3 HumanEval+ at the 4096-token-cap rerun. The first run at the 1024 default scored 74.4% — an artifact of un-disableable reasoning eating the token budget before code was generated. Details in [humaneval-plus.md](./test/test_humaneval-plus.md).

## Models tested

Each model that ran the full battery has its own page with results and notes:

| Model | Status | Role |
|---|---|---|
| [nous-deepseek-v4-flash](./models/nous-deepseek-v4-flash.md) | baseline | remote baseline |
| [qwen3.8-27b](./models/qwen3.8-27b.md) | kept | baseline, best coding |
| [qwen3.8-27b-heretic](./models/qwen3.8-27b-heretic.md) | kept | writing only |
| [GSQ-RCO-IQ3_S-mtp](./models/gsq-rco-iq3_s-mtp.md) | kept | footprint champion |
| [swift-qwen3.8-27b](./models/swift-qwen3.8-27b.md) | kept | default |
| [bonsai2](./models/bonsai2.md) | kept | tiny VRAM, 128K ctx |
| [ornith-1.5-35b-a3b](./models/ornith-1.5-35b-a3b.md) | kept | best coding |
| [exl3-qwen3.8-27b](./models/exl3-qwen3.8-27b.md) | engine comparison | same weights, different engine |
| [OrcaSAQ-2-27B](./models/orcasaq2-27b.md) | testing | hybrid attention, math leader |
| [crack2](./models/crack2.md) | experimental | weight-abliterated |
| [Signal-3.8-27B-AP](./models/signal-3.8-27b-ap.md) | rejected | mid on every axis |
| [gpt-oss-20b-mxfp4](./models/gpt-oss-20b-mxfp4.md) | rejected | smoke lied |
| [hermes-4.3-36b](./models/hermes-4.3-36b.md) | rejected | lost to 27B qwen pair |
| [qwen3-coder-30b-A3B](./models/qwen3-coder-30b-a3b.md) | rejected | coder that can't code |

Pruned models from smoke only (no full battery): Twin-Turbo, Signal-3.8-27B-AP, gemma-3-27b-it-qat, mistral-small-3.2-24b, Qwen-AgentWorld-35B-A3B, Muse-Glimmer-30B. See [smoke battery](./test/test_smoke-battery.md) for results.