# GPQA-Diamond

**TL;DR:** 198 graduate-level science MCQs. The hardest leg locally: best is Mirai S at 69.7%, most 27Bs land 44-56%. Chance is 25%. Thinking models need large budgets here or they score at the floor.

**What it evaluates:** 198 graduate-level science multiple-choice questions (biology, chemistry, physics). Expert knowledge + careful reasoning. Random guessing scores 25%. This is the hardest knowledge test in the battery.

**Reading scores:** 44-48% is the 27B-class norm on this hardware. 50%+ is exceptional. 40-43% is below par. Frontier models reach 60-80%. The remote flash baseline's 83.8% shows how far local knowledge still is from cloud — the gap is bigger here than on any other test.

## Results

| Model | [Origin](../README.md#results-summary) | Accuracy | Notes |
|---|---|---|---|
| nous-deepseek-v4-flash | China | 83.8% | remote baseline |
| **Nemotron 3.5 Bolt 74B-A9B** (IQ2_XXS, 9B active) | USA | **77.8%** | **best local** by 8 pts; 333.5 min. 57/198 items (29%) hit the 16,384 cap — 94.3% on the ones that finished |
| **Mirai S Qwen3.8-27B** (2.4 bpw trellis) | China | **69.7%** | 2nd local |
| **Muse Glimmer 30B** (Q4_K_M) | USA | **56.6%** | |
| Holo4-27B (Q4_K_M, Q8 KV) | China (FR pub.) | **56.1%** | |
| OrcaSAQ-2-27B (vLLM) [†](#fn-stuck) | China | ~55.0%\* | stopped at 20/198 — stuck-loop from reasoning; 71% null responses on first pass |
| **Nanbeige4.3-33B** (Q4_K_S) | China | 52.5% | 35.1 min |
| **Nemotron Cascade 2 30B A3B** (Q3_K_M) | USA | 54.0% | |
| swift-qwen3.8-27b | China | 50.5% | 136 average tokens (half of the next closest) |
| GSQ-RCO-IQ3_S-mtp | China (AT pub.) | 48.0% | |
| **Coder390-EfficientThink** (Q3LynnStyle-Q8MTP) | China | 48.0% / **87.1%**[±](#fn-coder390) | thinking-off / thinking-on (n=70 partial) |
| exl3-qwen3.8-27b [§](#fn-section) | China | 46.5% | thinking forced |
| qwen3.8-27b | China | 46.0% | |
| qwen3.8-27b-heretic | China | 44.4% | 1.8x qwen's tokens, lower score |
| bonsai2 (PrismML tern PTQ1_0) | China | 43.9% | |
| crack2 (abliterated PQ2_0) | China | 43.4% | |
| qwen3-coder-30b | China | 42.9% | |
| ornith-1.5-35b-a3b (q4_k_s) | not est. | **34.3%** | worst local; knowledge gap |
| Spark-X2.5-4B (Q4_K_M) | not est. | 25.3%[††](#fn-spark) | format artifact — 83% when content present |
| gemma-4-26b | USA | 15.0% | rejected |

**Scoring:** The model receives a free-form question with (A)-(D) options and is instructed to reply with `\boxed{letter}`. The parser extracts the LAST `\boxed{letter}` in the output (reasoning may contain boxed references; the final answer comes at the end).

\* Stopped early at 20/198 — stuck-loop from self-referential reasoning. ~55% is estimated from the truncated run plus the ~29% that produced valid responses on the first pass.

**Token budget:** Qwen3-based models (including the bonsai and swift variants) output reasoning inside `[think]...[/think]` tags before the answer. With a 1024-token cap, the model spends the entire budget on thinking and produces zero visible content — this affected 71% of responses in one early run. All GPQA figures here use a minimum 4096-token cap (8192 recommended for these architectures).


---

<a id="fn-stuck">†</a> Stuck-loop — model entered a self-referential reasoning loop; run stopped early.
<a id="fn-coder390">±</a> Coder390 GPQA is two runs, both qualified — 48.0% with thinking disabled (the comparable row), and 87.1% (n=70, 32K cap) with thinking enabled, killed at 70 items. A full 94K-budget rerun was abandoned at ~11 items: ~31K thinking tokens per question at ~31 tok/s projected 50+ hours. Details in [README](../README.md#fn-coder390).
<a id="fn-spark">††</a> Spark-X2.5-4B uses peg-native reasoning format — full thinking in `reasoning_content`, short answers in `content` only after thinking completes. On GPQA only 60/198 items produced content (83% accurate when they did). Details in [README](../README.md#fn-spark).
<a id="fn-section">§</a> Reasoning forced — model architecture cannot disable thinking.

**Hardware:** RTX 3090 24 GB, llama.cpp GGUF server (except EXL3 row), quantized models from HuggingFace.