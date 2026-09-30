# local-benchmarks / rtx-3090 — Local Model Benchmarks

**Last updated:** 2026-09-30

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
| **qwen3.8-27b** (GGUF Q5_K_M) | 86.8% | 91.5% | 77.3 / 83.5% | 46.0% | 88 | 18/20 | 1579 [‡](#fn-dagger) | baseline | | kept |
| **qwen3.8-27b-heretic** | 85.8% | 92.7% | 77.1 / 83.3% | 44.4% | 92 | 18/20 | 1631 [‡](#fn-dagger) | | 2/20 | kept, writing |
| **GSQ-RCO IQ3_S-mtp** | 85.4% | 91.5% | 79.3 / 85.0% | 48.0% | 88 | 18/20 | 1539 [‡](#fn-dagger) | | | kept, footprint |
| **swift-qwen3.8-27b** | **89.2%** | 87.8% | 79.5 / 85.1% | **50.5%** | 89 | **19/20** | 1479 [‡](#fn-dagger) | 1.41x | **20/20** | kept, default |
| **OrcaSAQ-2-27B** (vLLM) | **90.8%** | 89.6% | 76.0/82.2% [‡](#fn-dagger) | 55.0%*[†](#fn-stuck) | | | 1558[^](#fn-caret)[‡](#fn-dagger) | | | kept, math + long ctx |
| **Holo4-27B** (Q4_K_M, Q8 KV) | 73.4% | **90.2%** | 71.5/77.6% | **56.1%** | **91** | 9/20 | 1406 | 7/8 | 20/20 | tested |
| bonsai2 (PrismML tern PTQ1_0) | 85.6% | 89.0% | 74.3 / 81.8% | 43.9% | 87 | 18/20 | | | | kept: tiny VRAM, 128K ctx |
| ornith-1.5-35b-a3b (q4_k_s) | 85.2% | **94.5%** | 75.0 / 82.7% | 34.3% | 88 | **19/20** | | | 19/20 (95%) | best coding; worst GPQA; 64k ctx |
| exl3-qwen3.8-27b (3.5bpw) | 85.8% | 87.8%*[§](#fn-section) | 80.6 / 85.8%[§](#fn-section) | 46.5%[§](#fn-section) | 91[§](#fn-section) | | | | | EXL3 variant, reasoning forced |
| crack2 (PQ2_0 abliterate) | 83.6% | **76.2%** | 77.3 / 84.2% | 43.4% | 86 | **19/20** | | | | weight-abliterated, HE+ collapse |
| **Spark-X2.5-4B** (Q4_K_M) | 72.2% | 70.7% | 68.6 / 75.3% | 25.3%[††](#fn-spark) | 83 | 0/20[††](#fn-spark) | 1500[††](#fn-spark) | 0/8[††](#fn-spark) | — | 4B agent model, 185 tok/s, 6.3 GB VRAM. Scores marked †† depressed by reasoning-content format mismatch |

\* EXL3 HumanEval+ at the 4096-token-cap rerun. The first run at the 1024 default scored 74.4% — an artifact of un-disableable reasoning eating the token budget before code was generated. Details in [humaneval-plus.md](./test/test_humaneval-plus.md).

† OrcaSAQ-2-27B GPQA-Diamond at 20/198 (55.0%*) — stopped early. The model's self-feedback loop consumed output tokens on thinking, producing null responses on 71% of first-attempt prompts. Score at 20 items is partial, not comparable.

‡ Tested with `--no-think` (reasoning disabled). Thinking degrades or stalls on these tasks — details in the [Think vs No-Think](#think-vs-no-think) section below.

§ EXL3 engine forces reasoning with no off switch. All results reflect thinking-enabled mode — not directly comparable to llama.cpp GGUF runs of the same weights.

^ OrcaSAQ-2-27B Prose ELO via LiteLLM proxy with `chat_template_kwargs: {enable_thinking: false}` (API-level equivalent of `--no-think`).

†† Spark-X2.5-4B uses peg-native reasoning format: the model outputs full thinking in `reasoning_content` and only produces short answers in `content` after thinking completes. Bench scripts read `content` and score empty-as-fail. On GPQA, only 60/198 items produced content (83% accuracy when they did). On Instr v2, Reasoning bench, and 6/10 Prose tasks, every item exhausted its token budget before reaching the answer. IFEval at 68.6/75.3% is depressed for the same reason (1.4M tokens burned on thinking). MATH, HumanEval+, and Toolbench (83) are unaffected — the thinking completes within the default budget on those tasks, so those scores are genuine.

## Think vs No-Think

These models use Qwen3's built-in reasoning (thinking tokens inside `[think]...[/think]`). Whether thinking helps or hurts depends on the task:

| Task | Thinking | No-Thinking | Why |
|---|---|---|---|
| **MATH-500** | strongest | — | Multi-step reasoning benefits from explicit chain-of-thought |
| **HumanEval+** | comparable | comparable | Coding benefits are marginal; token budget matters more |
| **IFEval** | stall-prone | stable | Thinking produces self-feedback loops on mechanical constraints |
| **Prose ELO** | null output | full answer | Thinking consumes all tokens before visible content |
| **GPQA-Diamond** | higher score* | — | Hard science benefits from reasoning, but stalls are common |

\* OrcaSAQ-2-27B's 55.0%*† at 20/198 is not statistically comparable to other models' full 198-item runs.

For the full battery, models run in their default config (thinking on). Tests that stall or degrade (IFEval, Prose ELO) are retried with `--no-think`. Results in the summary table carry config markers so you can compare apples to apples.

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
| [OrcaSAQ-2-27B](./models/orcasaq2-27b.md) | kept | math leader, 262K context |
| [Holo4-27B](./models/holo4-27b.md) | tested | GPQA leader, strong coding, weak math |
| [crack2](./models/crack2.md) | experimental | weight-abliterated |

---

<a id="fn-dagger">‡</a> `--no-think` mode.
<a id="fn-stuck">†</a> Stuck-loop — model entered a self-referential reasoning loop; run stopped early.
<a id="fn-section">§</a> Reasoning forced — model architecture cannot disable thinking.
<a id="fn-caret">^</a> LiteLLM proxy — served through the LiteLLM proxy instead of direct inference.
