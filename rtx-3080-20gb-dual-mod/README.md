# local-benchmarks / rtx-3080-20gb-dual-mod — Local Model Benchmarks

**Last updated:** 2026-11-08

## TL;DR

**Qwen3.8-Flash-Next (GSQ-RCO IQ3_XXS) is the first full battery on this box** — a 125B-parameter MoE with 6B active, served by Strata with DFlash2 speculative decode at 64K context. It runs the whole 13-leg battery in under four hours of wall time at **~105–165 tok/s**, and its strong legs are the ones that matter for agentic use: **MMLU 90.8%** (454/500), **LiveCodeBench 62.0% on a full 100-problem run**, **tool-eval 90/100 ★★★★★ with the safety gate passed**, Instruction v2 20/20, reasoning bench 8/8, and a **perfect harmful-refusal profile (20/20)** with low over-refusal (4/30 benign). HermesAgent-20 lands at 13/20 pass@1 (**14/20 or 70% best-of-3**), zero runaway loops, 469,498 tokens — a lean agentic profile.

Two honest caveats. **GPQA-Diamond 50.0%** (99/198) is mid-pack for a Qwen3-class base — science knowledge is not this model's edge. **IFBench is weak**: strict/loose prompt accuracy 28.0/30.7%, strict/loose instruction 28.8/32.6% — the mechanical-formatting leg, not the reasoning legs. Details in [its model page](./models/qwen3.8-flash-next-gsq-rco.md).

The whole battery ran with thinking off (this model's default here). The one leg whose pack defaults to thinking-on — HermesAgent-20 — was forced off to stay consistent, flagged with [‡](#fn-dagger) below.

## System

| Component | Detail |
|---|---|
| GPU | 2× NVIDIA GeForce RTX 3080 20 GB (modded, no ECC) |
| CPU | 2× Intel Xeon Gold 6138 @ 2.00 GHz (20 cores / 40 threads each, 80 threads total) |
| RAM | 262 GiB |
| OS | Ubuntu 24.04.4 LTS |
| Inference engine | Strata 0.1.57 (CUDA 13) + DFlash2 speculative decode |
| Model pack | Qwen3.8-Flash-Next IQ3_XXS (75.8 GB), 125B MoE / 6B active, 64K ctx, int8 KV |

The model spans both cards (~19.4 GB VRAM each). No ComfyUI-style shared-VRAM budget applies on this box; nothing else runs on the cards during a battery.

## What's tested

Each benchmark evaluates a specific capability. The harness and conventions match the [rtx-3090 section](../rtx-3090/README.md); scoring code lives in the same repo.

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
| [MMLU](./test/test_mmlu.md) | Multi-subject knowledge retention across 57 subjects | 500 |
| [LiveCodeBench](./test/test_livecodebench.md) | Competition coding with hidden tests, contamination-free window | 100 |
| [HermesAgent-20](./test/test_hermesagent-20.md) | Agentic harness behavior — drives a real agent runtime in Docker and grades the artifacts it produces | 20 scenarios |

## Results summary

Blank cells mean the model was pruned before running that test. **Origin** is the country of the model's **base weights** — the lab whose pretraining produced them, not the quantizer: this model's base is Alibaba Qwen (China); the GSQ-RCO quantization is by ISTA-DASLab (Austria), marked `China (AT pub.)`.

| Model | Origin | MATH-500 | HumanEval+ | IFEval P/I | GPQA-D | Tool-eval | Instr v2 | Reasoning | Refusal H | MMLU | LiveCodeBench | HermesAgent-20 | Status |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| **Qwen3.8-Flash-Next** (IQ3_XXS, 6B active) | China (AT pub.) | 83.2% | 82.9% | 80.4 / 85.8%[◊](#fn-ifeval) | 50.0% | **90** (★★★★★) | **20/20** | 8/8 | **20/20** | **90.8%** | **62.0%** | 14/20 (70%) pass@k[‡](#fn-dagger) | tested — first full battery on this box; clean tool safety gate |

Thinking-off was the run config for every leg. For Qwen3-family reasoning models the thinking/no-thinking tradeoff differs by task:

| Task | Thinking | No-Thinking | Why |
|---|---|---|---|
| **MATH-500** | strongest | — | Multi-step reasoning benefits from explicit chain-of-thought |
| **HumanEval+** | comparable | comparable | Coding benefits are marginal; token budget matters more |
| **IFEval** | stall-prone | stable | Thinking produces self-feedback loops on mechanical constraints |

---

<a id="fn-dagger">‡</a> HermesAgent-20 was run with `--no-thinking` (thinking_mode `force-off` in the result JSON). The pack defaults to thinking-enabled; forcing it off matches the rest of this battery. 13/20 pass@1, 14/20 best-of-3, zero runaway loops, 469,498 tokens.
<a id="fn-ifeval">◊</a> IFEval 80.4 / 85.8% (435/541, 716/834) read from the battery driver log (32.3 min, 105.4 tok/s). The harness wrote no separate IFEval artifact this run — same quirk documented in the [rtx-3090 section](../rtx-3090/README.md#fn-arete-if).
