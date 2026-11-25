# Models

Every model tested, one row each, with the hardware it was tested on. Hardware-first views live in [rtx-3090](../rtx-3090/README.md) and [rtx-3080-20gb-dual-mod](../rtx-3080-20gb-dual-mod/README.md). Per-model pages carry the full results, timing, and caveats.

Scores are not always comparable across rows. Different hardware, engines, context sizes and thinking settings are involved, and some rows carry footnotes that say so. Read the footnote links before ranking.

Blank cells mean the model was pruned before running that test. **Origin** is the country of the model's base weights.

| Model | Origin | Hardware | MATH-500 | HumanEval+ | IFEval P/I | GPQA-D | Tool-eval | Instr v2 | Reasoning | Refusal H | MMLU | LiveCodeBench | HermesAgent-20 | Status |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| [**Arete-27B** (Q4_K_M)](./arete-27b.md) | China (US pub.) | RTX 3090 24 GB | 90.4% | 94.5% | 78.6 / 85.0%[◊](../rtx-3090/README.md#fn-arete-if) | 55.6% | 89 | 19/20 | 8/8 | **6/20** | **88.4%** | —[~](../rtx-3090/README.md#fn-arete-lcb) | 17/20 | tested: **MMLU leader**; best HermesAgent-20; **refusal FAIL** (6/20 harmful); LCB killed at 75/100 |
| [bonsai2 (PrismML tern PTQ1_0)](./bonsai2.md) | China | RTX 3090 24 GB | 85.6% | 89.0% | 74.3 / 81.8% | 43.9% | 87 | 18/20 |  |  |  |  |  | kept: tiny VRAM, 128K ctx |
| [**Coder390-EfficientThink** (Q3LynnStyle-Q8MTP)](./coder390-efficientthink.md) | China | RTX 3090 24 GB | 89.0% | 79.9% | 71.5 / 84.9% | 48.0% / **87.1%**[±](../rtx-3090/README.md#fn-coder390) |  | **20/20** |  |  |  |  |  | pruned: thinking model, unusable pace at Q3 on 24 GB |
| [crack2 (PQ2_0 abliterate)](./crack2.md) | China | RTX 3090 24 GB | 83.6% | **76.2%** | 77.3 / 84.2% | 43.4% | 86 | **19/20** |  |  |  |  |  | weight-abliterated, HE+ collapse |
| [exl3-qwen3.8-27b (3.5bpw)](./exl3-qwen3.8-27b.md) | China | RTX 3090 24 GB | 85.8% | 87.8%*[§](../rtx-3090/README.md#fn-section) | 80.6 / 85.8%[§](../rtx-3090/README.md#fn-section) | 46.5%[§](../rtx-3090/README.md#fn-section) | 91[§](../rtx-3090/README.md#fn-section) |  |  |  |  |  |  | EXL3 variant, reasoning forced |
| gemma-3-27b-it-qat | USA | RTX 3090 24 GB |  |  |  |  |  |  |  |  |  |  |  | pruned (smoke: tool 0/10) |
| [gemma-4-26b](./gemma-4-26b.md) | USA | RTX 3090 24 GB | 48.8% | 56.1% |  | 15.0% |  |  |  |  |  |  |  | rejected — far below field |
| [GLM-5.3-Flash-Maya-S24](./glm-5.3-flash-maya-s24.md) | China | RTX 3090 24 GB |  |  |  |  |  |  |  |  |  |  |  | speed probe only: 4.5 tok/s decode, no battery run |
| [gpt-oss-20b-mxfp4](./gpt-oss-20b-mxfp4.md) | USA | RTX 3090 24 GB | 74.2% | 67.7% |  |  | 75 |  |  |  |  |  |  | pruned |
| [**GSQ-RCO IQ3_S-mtp**](./gsq-rco-iq3_s-mtp.md) | China (AT pub.) | RTX 3090 24 GB | 85.4% | 91.5% | 79.3 / 85.0% | 48.0% | 88 | 18/20 |  |  |  |  |  | kept, footprint |
| [hermes-4.3-36b](./hermes-4.3-36b.md) | USA | RTX 3090 24 GB | 81.0% | 87.2% |  |  |  |  |  |  |  |  |  | pruned |
| [**Holo4-27B** (Q4_K_M, Q8 KV)](./holo4-27b.md) | China (FR pub.) | RTX 3090 24 GB | 73.4% | **90.2%** | 71.5/77.6% | **56.1%** | **91** | 9/20 | 7/8 | 20/20 |  |  |  | tested |
| [HyperQwen (Qwen3.8-27B + DFlash2)](./hyperqwen.md) | China | RTX 3090 24 GB |  |  |  |  |  |  |  |  |  |  |  | rejected: results were mislabeled files from other models |
| [**Inkling 2 Mini** (UD-Q4_K_XL, Project Maya)](./inkling-2-mini-maya.md) | USA | RTX 3090 24 GB | 95.6%[※※](../rtx-3090/README.md#fn-inkling-math) | 92.7% | 78.7 / 83.0% | 79.3%[‡‡](../rtx-3090/README.md#fn-inkling-gpqa) | 87[◊◊](../rtx-3090/README.md#fn-inkling-tool) | 20/20 | 8/8 | 19/20 | 76.2% | —[~~](../rtx-3090/README.md#fn-inkling-lcb) | 14/20 | tested: 173 tok/s decode on Maya, 74.6 tok/s at 237K context ([depth test](../rtx-3090/test/test_context-depth-speed.md)); MATH-500 is a regrade, see footnote; [model page](inkling-2-mini-maya.md) |
| [**LFM2.5-24B-A2B** (Q4_K_M, 2B active MoE)](./lfm2.5-24b-a2b.md) | USA | RTX 3090 24 GB | 76.4% | 91.5% | **84.3 / 89.0%** | 50.5% | 84 [◊](../rtx-3090/README.md#fn-lfm) | 15/20 | 8/8 | 20/20 | 73.6% | 61.0% |  | tested: **191 tok/s speed champ**, best IFEval; **tool safety gate failed** |
| [**Mirai S Qwen3.8-27B** (2.4 bpw trellis)](./mirai-s-qwen3-8-27b.md) | China | RTX 3090 24 GB | **94.2%** | **95.1%** | 78.9 / 85.1% | 69.7% | **93** | **20/20** |  | 20/20 | 87.0% | **76.0%**[※](../rtx-3090/README.md#fn-lcb) |  | kept, best all-round; instruction + agentic leader |
| mistral-small-3.2-24b | France | RTX 3090 24 GB |  |  |  |  |  |  |  |  |  |  |  | pruned (smoke: reason 3/10) |
| [**Muse Glimmer 30B** (Q4_K_M)](./muse-glimmer-30b.md) | USA | RTX 3090 24 GB | 82.6% | 62.2% |  | **56.6%** |  | **15/20** |  |  |  |  |  | tested: strong writer, solid GPQA, weak coder |
| [**Nanbeige4.3-33B** (Q4_K_S)](./nanbeige4.3-33b.md) | China | RTX 3090 24 GB | 83.2% | 68.3% | not scored[~](../rtx-3090/README.md#fn-nb-if) | 52.5% | 88[◊](../rtx-3090/README.md#fn-nb-gate) | 17/20 | 8/8 | 20/20 | 85.6% | —[◆](../rtx-3090/README.md#fn-nb-lcb) | 15/20 | rejected: [model page](nanbeige4.3-33b.md) — dominated by Mirai S on every scored axis; vendor agentic claim not reproduced |
| [**Nemotron 3.5 Bolt 74B-A9B** (IQ2_XXS, 9B active)](./nemotron-3.5-bolt-74b-a9b.md) | USA | RTX 3090 24 GB | **96.4%** | **96.3%** | 76.2 / 83.3%[‡](../rtx-3090/README.md#fn-dagger) | **77.8%** | 91[◆](../rtx-3090/README.md#fn-bolt) | 19/20 |  |  | 88.2%[※](../rtx-3090/README.md#fn-bolt-mmlu) | —[◆](../rtx-3090/README.md#fn-bolt) |  | tested: leads 3 of 8 legs (MMLU ceded to Arete); **tool safety gate failed**; LCB killed at 54/100 |
| [**Nemotron Cascade 2 30B A3B** (Q3_K_M)](./nemotron-cascade-2-30b.md) | USA | RTX 3090 24 GB | 81.2% | 84.8% |  | 54.0% |  | 9/20[¶](../rtx-3090/README.md#fn-para) |  |  |  |  |  | tested: thinking model, fast, needs high budget |
| [nous-deepseek-v4-flash (remote)](./nous-deepseek-v4-flash.md) | China | RTX 3090 24 GB | 98.2% | 93.9% | 86.7 / 90.9% | 83.8% | 93 |  |  |  |  |  |  | remote baseline |
| [**OrcaSAQ-2-27B** (vLLM)](./orcasaq2-27b.md) | China | RTX 3090 24 GB | **90.8%** | 89.6% | 76.0/82.2% [‡](../rtx-3090/README.md#fn-dagger) | 43.9%[†](../rtx-3090/README.md#fn-stuck) |  |  |  |  |  |  |  | kept, math + long ctx |
| [ornith-1.5-35b-a3b (q4_k_s)](./ornith-1.5-35b-a3b.md) | not est. | RTX 3090 24 GB | 85.2% | **94.5%** | 75.0 / 82.7% | 34.3% | 88 | **19/20** |  | 19/20 (95%) |  |  |  | 2nd coding; worst GPQA; 64k ctx |
| [**qwen3.8-27b** (GGUF Q5_K_M)](./qwen3.8-27b.md) | China | RTX 3090 24 GB | 86.8% | 91.5% | 77.3 / 83.5% | 46.0% | 88 | 18/20 | baseline |  |  |  |  | kept |
| [**qwen3.8-27b-heretic**](./qwen3.8-27b-heretic.md) | China | RTX 3090 24 GB | 85.8% | 92.7% | 77.1 / 83.3% | 44.4% | 92 | 18/20 |  | 2/20 |  |  |  | kept, writing |
| [**Qwen3.8-27B-TurboFCFusion** ("turbo-fable", Q4_K_M)](./qwen3.8-27b-turbofcfusion.md) | China | RTX 3090 24 GB | 81.0% | **86.0%** |  | 35.9% |  | **19/20** |  |  |  |  |  | tested: strong coder, slow, token-hungry; full page [here](qwen3.8-27b-turbofcfusion.md) |
| [**Qwen3.8-Flash-Next** (IQ3_XXS, 6B active)](./qwen3.8-flash-next-gsq-rco.md) | China (AT pub.) | 2× RTX 3080 20 GB (modded) | **91.0%**[†](../rtx-3080-20gb-dual-mod/README.md#fn-thinkon) | 82.9% | 80.4 / 85.8%[◊](../rtx-3080-20gb-dual-mod/README.md#fn-ifeval) | **78.3%**[†](../rtx-3080-20gb-dual-mod/README.md#fn-thinkon) | **90** (★★★★★) | **20/20** | 8/8 | **20/20** | **91.0%**[†](../rtx-3080-20gb-dual-mod/README.md#fn-thinkon) | **62.0%** | 14/20 (70%) pass@k[‡](../rtx-3080-20gb-dual-mod/README.md#fn-dagger) | tested — first full battery on this box; clean tool safety gate; reasoning legs rerun thinking-on |
| [qwen3-coder-30b-A3B](./qwen3-coder-30b-a3b.md) | China | RTX 3090 24 GB | 85.0% | 73.2% | 74.3 / 82.0% | 42.9% | 68 |  |  |  |  |  |  | pruned |
| Qwen-AgentWorld-35B-A3B | China | RTX 3090 24 GB |  |  |  |  |  |  |  |  |  |  |  | pruned (smoke: reason 4/10) |
| [Signal-3.8-27B-AP](./signal-3.8-27b-ap.md) | China | RTX 3090 24 GB | 85.6% | 84.1% |  |  | 83 |  |  |  |  |  |  | pruned |
| [**Spark-X2.5-4B** (Q4_K_M)](./spark-x2.5-4b.md) | not est. | RTX 3090 24 GB | 72.2% | 70.7% | 68.6 / 75.3% | 25.3%[††](../rtx-3090/README.md#fn-spark) | 83 | 3/20[††](../rtx-3090/README.md#fn-spark) | 8/8[†](../rtx-3090/README.md#fn-stuck) | — |  |  |  | 4B agent model, 185 tok/s, 6.3 GB VRAM. Scores marked †† depressed by reasoning-content format mismatch |
| [**swift-qwen3.8-27b**](./swift-qwen3.8-27b.md) | China | RTX 3090 24 GB | **89.2%** | 87.8% | 79.5 / 85.1% | **50.5%** | 89 | **19/20** | 1.41x | **20/20** |  |  |  | kept, capable all-rounder |
| Twin-Turbo | not est. | RTX 3090 24 GB |  |  |  |  |  |  | 0.59x |  |  |  |  | pruned |
