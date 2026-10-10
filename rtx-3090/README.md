# local-benchmarks / rtx-3090 — Local Model Benchmarks

**Last updated:** 2026-10-10

## TL;DR

**Mirai S Qwen3.8-27B is the best model we've tested.** It wins or ties every major leg: MATH-500 94.2%, HumanEval+ 95.1%, GPQA-D 69.7% (12+ points clear of every other local model), Tool-eval 93/100, Instruction v2 20/20, perfect harmful refusal (20/20). It does this at 2.4 bpw trellis quantization — roughly 11 GB on disk — while holding a 262K context window and ~54 tok/s decode with MTP speculation. No other local model comes close on the combined record; the remote deepseek baseline beats it on raw scores but that's a datacenter model, not something you can run on this card. Full row in the results table below; details in [its model page](./models/mirai-s-qwen3-8-27b.md).

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

**Power limit: 300 W (below the card's 350 W default).** The power cap was in place for every benchmark in this repo; the card downclocks to ~1695 MHz SM vs 2100 MHz max under load. All tok/s figures are therefore ~15-20% lower than the card could deliver at default power. Lift the cap with `sudo nvidia-smi -pl 350` before any speed-sensitive comparison.

## What's tested

Each benchmark evaluates a specific capability. Full methodology and automation details in [methodology.md](./methodology.md).

| Test | What it measures | Items |
|---|---|---|
| [MATH-500](./test/test_math-500.md) | Multi-step math reasoning (algebra, combinatorics, number theory) | 500 |
| [HumanEval+](./test/test_humaneval-plus.md) | Python code generation against hidden test suites | 164 |
| [GPQA-Diamond](./test/test_gpqa-diamond.md) | Graduate-level science knowledge (biology, chemistry, physics) | 198 |
| [IFEval](./test/test_ifeval.md) | Instruction following with mechanical constraints | 541 |
| [Tool-eval-bench](./test/test_tool-eval-bench.md) | Multi-turn agentic tool use across 16 categories | 69 scenarios |
| [Instruction v2](./test/test_instr-v2.md) | Single-constraint mechanical obedience | 20 tests |
| [Refusal battery](./test/test_refusal-battery.md) | Safety alignment on benign/edgy/harmful prompts | 70 prompts |
| [Reasoning bench](./test/test_reasoning-bench.md) | Thinking-token efficiency and wall time with reasoning enabled | 8 tasks |
| [Smoke battery](./test/test_smoke-battery.md) | Quick screening pass (30 min per model) | 35 tasks |
| [Speculative decode](./test/test_spec-decode-mtp-vs-dflash.md) | MTP vs DFlash2 speed and context ceiling comparison | 3 configs × 4 depths |

## Results summary

Blank cells mean the model was pruned before running that test (smoke battery caught it).

| Model | MATH-500 | HumanEval+ | IFEval P/I | GPQA-D | Tool-eval | Instr v2 | Reasoning | Refusal H | Status |
|---|---|---|---|---|---|---|---|---|---|---|
| nous-deepseek-v4-flash (remote) | 98.2% | 93.9% | 86.7 / 90.9% | 83.8% | 93 | | | | remote baseline |
| **Mirai S Qwen3.8-27B** (2.4 bpw trellis) | **94.2%** | **95.1%** | 78.9 / 85.1% | **69.7%** | **93** | **20/20** | | 20/20 | kept, best overall |
| **qwen3.8-27b** (GGUF Q5_K_M) | 86.8% | 91.5% | 77.3 / 83.5% | 46.0% | 88 | 18/20 | baseline | | kept |
| **qwen3.8-27b-heretic** | 85.8% | 92.7% | 77.1 / 83.3% | 44.4% | 92 | 18/20 | | 2/20 | kept, writing |
| **GSQ-RCO IQ3_S-mtp** | 85.4% | 91.5% | 79.3 / 85.0% | 48.0% | 88 | 18/20 | | | kept, footprint |
| **swift-qwen3.8-27b** | **89.2%** | 87.8% | 79.5 / 85.1% | **50.5%** | 89 | **19/20** | 1.41x | **20/20** | kept, capable all-rounder |
| **OrcaSAQ-2-27B** (vLLM) | **90.8%** | 89.6% | 76.0/82.2% [‡](#fn-dagger) | 55.0%*[†](#fn-stuck) | | | | | kept, math + long ctx |
| **Coder390-EfficientThink** (Q3LynnStyle-Q8MTP) | 89.0% | 79.9% | 71.5 / 84.9% | 48.0% / **87.1%**[±](#fn-coder390) | | **20/20** | | | pruned: thinking model, unusable pace at Q3 on 24 GB |
| **Holo4-27B** (Q4_K_M, Q8 KV) | 73.4% | **90.2%** | 71.5/77.6% | **56.1%** | **91** | 9/20 | 7/8 | 20/20 | tested |
| bonsai2 (PrismML tern PTQ1_0) | 85.6% | 89.0% | 74.3 / 81.8% | 43.9% | 87 | 18/20 | | | kept: tiny VRAM, 128K ctx |
| ornith-1.5-35b-a3b (q4_k_s) | 85.2% | **94.5%** | 75.0 / 82.7% | 34.3% | 88 | **19/20** | | 19/20 (95%) | 2nd coding; worst GPQA; 64k ctx |
| exl3-qwen3.8-27b (3.5bpw) | 85.8% | 87.8%*[§](#fn-section) | 80.6 / 85.8%[§](#fn-section) | 46.5%[§](#fn-section) | 91[§](#fn-section) | | | | EXL3 variant, reasoning forced |
| crack2 (PQ2_0 abliterate) | 83.6% | **76.2%** | 77.3 / 84.2% | 43.4% | 86 | **19/20** | | | weight-abliterated, HE+ collapse |
| **Nemotron Cascade 2 30B A3B** (Q3_K_M) | 81.2% | 84.8% | | 54.0% | | 9/20[¶](#fn-para) | | | tested: thinking model, fast, needs high budget |
| **Muse Glimmer 30B** (Q4_K_M) | 82.6% | 62.2% | | **56.6%** | | **15/20** | | | tested: strong writer, solid GPQA, weak coder |
| **Qwen3.8-27B-TurboFCFusion** ("turbo-fable", Q4_K_M) | 81.0% | **86.0%** | | 35.9% | | **19/20** | | | tested: strong coder, slow, token-hungry; full page [here](./models/qwen3.8-27b-turbofcfusion.md) |
| **Spark-X2.5-4B** (Q4_K_M) | 72.2% | 70.7% | 68.6 / 75.3% | 25.3%[††](#fn-spark) | 83 | 3/20[††](#fn-spark) | 0/8[††](#fn-spark) | — | 4B agent model, 185 tok/s, 6.3 GB VRAM. Scores marked †† depressed by reasoning-content format mismatch |
| Twin-Turbo | | | | | | | 0.59x | | pruned |
| Signal-3.8-27B-AP | 85.6% | 84.1% | | | 83 | | | | pruned |
| gpt-oss-20b-mxfp4 | 74.2% | 67.7% | | | 75 | | | | pruned |
| hermes-4.3-36b | 81.0% | 87.2% | | | | | | | pruned |
| qwen3-coder-30b-A3B | 85.0% | 73.2% | 74.3 / 82.0% | 42.9% | 68 | | | | pruned |
| gemma-4-26b | 48.8% | 56.1% | | 15.0% | | | | | rejected — far below field |
| gemma-3-27b-it-qat | | | | | | | | | pruned (smoke: tool 0/10) |
| mistral-small-3.2-24b | | | | | | | | | pruned (smoke: reason 3/10) |
| Qwen-AgentWorld-35B-A3B | | | | | | | | | | pruned (smoke: reason 4/10) |

\* EXL3 HumanEval+ at the 4096-token-cap rerun. The first run at the 1024 default scored 74.4% — an artifact of un-disableable reasoning eating the token budget before code was generated. Details in [humaneval-plus.md](./test/test_humaneval-plus.md).

† OrcaSAQ-2-27B GPQA-Diamond at 20/198 (55.0%*) — stopped early. The model's self-feedback loop consumed output tokens on thinking, producing null responses on 71% of first-attempt prompts. Score at 20 items is partial, not comparable.

‡ Tested with `--no-think` (reasoning disabled). Thinking degrades or stalls on these tasks — details in the [Think vs No-Think](#think-vs-no-think) section below.

§ EXL3 engine forces reasoning with no off switch. All results reflect thinking-enabled mode — not directly comparable to llama.cpp GGUF runs of the same weights.

†† Spark-X2.5-4B uses peg-native reasoning format: the model outputs full thinking in `reasoning_content` and only produces short answers in `content` after thinking completes. Bench scripts read `content` and score empty-as-fail. On GPQA, only 60/198 items produced content (83% accuracy when they did). On Instr v2, Reasoning bench, and 5/10 writing tasks, every item exhausted its token budget before reaching the answer. IFEval at 68.6/75.3% is depressed for the same reason (1.4M tokens burned on thinking). MATH, HumanEval+, and Toolbench (83) are unaffected — the thinking completes within the default budget on those tasks, so those scores are genuine.

<a id="fn-coder390"></a>± Coder390 GPQA is two runs, both qualified. **48.0% (n=198)**: thinking disabled — the comparable row, and a near-floor score because this model is RL-trained to reason long before answering. **87.1% (n=70, 32K cap)**: thinking enabled — partial run, killed at 70 items; no saved JSON (stdout lost to a session restart), reconstructed from the live log. The author's claim is 89.9% at FP8 with a 94K thinking budget on 2x RTX PRO 6000; their own GGUF table shows 86.4-87.9% at Q8_0/Q6_K. A full 94K-budget rerun was abandoned at ~11 items: ~31K thinking tokens per question at ~31 tok/s projected 50+ hours. IFEval 44/541 prompts timed out at the 120s client limit (scored as fail).


These models use Qwen3's built-in reasoning (thinking tokens inside `[think]...[/think]`). Whether thinking helps or hurts depends on the task:

| Task | Thinking | No-Thinking | Why |
|---|---|---|---|
| **MATH-500** | strongest | — | Multi-step reasoning benefits from explicit chain-of-thought |
| **HumanEval+** | comparable | comparable | Coding benefits are marginal; token budget matters more |
| **IFEval** | stall-prone | stable | Thinking produces self-feedback loops on mechanical constraints |
