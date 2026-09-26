# Prose ELO (Writing Quality)

**What it evaluates:** 10 writing tasks per model (product blurbs, email replies, blog intros), then a judge model compares outputs pairwise and rates them ELO-style on clarity, style, persuasiveness, adherence, and fluff. This test exists because reasoning quality and writing quality are NOT correlated on this hardware: the best reasoner (gpt-oss) was the worst writer, and verbosity is baked into some models' weights — system prompts don't fix it.

**How it's scored:** Outputs are judged pairwise by **nous-deepseek-v4-flash** — 150 judgments (15 pairs x 10 tasks), anonymized prompt (the judge sees "Sample A / Sample B", never model names), randomized side assignment to cancel position bias, ELO k=32, anchor 1500.

**Reading scores:** 1500 is the anchor. Above 1600 is a clear winner. Below 1400 is a real loss. Judge choice matters: a different judge can move the order — treat any single-judge ELO as judge-relative, not absolute quality.

## Results

| Model | ELO | W-L-T |
|---|---|---|
| **qwen3.8-27b-heretic** | **1631** | 33-16-1 |
| **qwen3.8-27b** | **1579** | 32-18-0 |
| GSQ-RCO-IQ3_S-mtp | 1539 | 25-24-1 |
| swift-qwen3.8-27b | 1479 | 23-27-0 |
| hermes-4.3-36b | 1440 | 18-32-0 |
| Muse-Glimmer-30B | 1332 | 18-32-0 |

Head-to-head highlights: heretic beats swift 7-3 and GSQ 8-1 but loses to qwen 7-3; qwen beats everyone except a 5-5 tie with Muse; GSQ beats swift 8-2; swift's only dominant win is Muse 9-1.

Caveats: one judge, one pass, single temperature.

**Hardware:** RTX 3090 24 GB, llama.cpp GGUF server for output generation; remote frontier model for judging.