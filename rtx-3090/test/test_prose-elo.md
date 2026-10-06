# Prose ELO (Writing Quality)

**What it evaluates:** 10 writing tasks per model (product blurbs, email replies, blog intros), then a judge model compares outputs pairwise and rates them ELO-style on clarity, style, persuasiveness, adherence, and fluff. This test exists because reasoning quality and writing quality are NOT correlated on this hardware: the best reasoner (gpt-oss) was the worst writer, and verbosity is baked into some models' weights — system prompts don't fix it.

**How it's scored:** Outputs are judged pairwise by **nous-deepseek-v4-flash** — 150 judgments (15 pairs x 10 tasks), anonymized prompt (the judge sees "Sample A / Sample B", never model names), randomized side assignment to cancel position bias, ELO k=32, anchor 1500.

**Reading scores:** 1500 is the anchor. Above 1600 is a clear winner. Below 1400 is a real loss. Judge choice matters: a different judge can move the order — treat any single-judge ELO as judge-relative, not absolute quality.

## Results

| Model | ELO | W-L-T | Notes |
|---|---|---|---|
| **Qwen3.8-27B-GSQ-RCO-IQ3_S-mtp** | **1733** [‡](#fn-dagger) | 35-16-9 | Best overall prose, consistent across genres |
| **swift-qwen3.8-27b** | **1694** [‡](#fn-dagger) | 31-20-9 | Strong, broad, underrated by earlier passes |
| **qwen3.8-27b** | **1681** [‡](#fn-dagger) | 30-21-9 | Solid prose, trails heretic/GSQ narrowly |
| **qwen3.8-27b-heretic** | **1603** [‡](#fn-dagger) | 28-23-9 | Good but slightly below base Qwen in head-to-head |
| **OrcaSAQ-2-27B** (vLLM) | **1558** [^](#fn-caret)[‡](#fn-dagger) | 4-0-16* | Separate pass — undefeated but mostly ties |
| **Muse Glimmer 30B** | **1532** [¶](#fn-para) | 27-24-9 | Above baseline, ~40% wins, best writing of models tested in 2026-10 |
| **Holo4-27B** | **1406** [‡](#fn-dagger) | 18-32-0 | Mid — below Qwen family baseline |
| **hermes-4.3-36b** | **1214** | 15-35-10 | Below baseline, clearly outclassed |
| **Nemotron Cascade 2 30B A3B** | **1043** [¶](#fn-para) | 12-37-11 | Floor — 8/10 tasks produced empty content within 1500-token budget |

Head-to-head highlights: heretic beats swift 7-3 and GSQ 8-1 but loses to qwen 7-3; GSQ beats swift 8-2 and Muse 5-1; Muse beats Nemotron 7-0 and hermes 8-2 but loses to GSQ 5-1 and swift 5-2.

\* OrcaSAQ-2-27B judged in a separate pass against qwen3.8-27b and hermes-4.3-36b-reasoning (nous-deepseek-v4-flash judge). Undefeated vs both baselines. W-L-T of 4-0-16 means 4 wins, 0 losses, 16 ties across 20 comparisons (10 tasks × 2 baselines) — beat or matched every comparison, never lost. ELO is not directly comparable to the main battery (different judge session). 80% ties suggests the judge found the outputs hard to distinguish, not that OrcaSAQ2 narrowly scraped by.

Caveats: one judge, one pass, single temperature. ELO recomputed 2026-10-06 with draw-excluded win-rate formula — values differ from earlier table (draw-inclusive) but ordering is consistent.


---

<a id="fn-caret">^</a> LiteLLM proxy — served through the LiteLLM proxy instead of direct inference.
<a id="fn-dagger">‡</a> `--no-think` mode.
<a id="fn-para">¶</a> Reasoning-content format mismatch — model outputs full thinking in `reasoning_content` and only short answers in `content`. At standard token budgets (400-1500), the model burns most tokens on reasoning. Scores reflect the constrained budget, not the model's ceiling.

**Hardware:** RTX 3090 24 GB, llama.cpp GGUF server for output generation; remote frontier model for judging.