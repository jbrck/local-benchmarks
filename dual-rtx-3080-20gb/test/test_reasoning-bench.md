# Reasoning Bench (Thinking-Token Efficiency)

**TL;DR:** 8 reasoning tasks with reasoning enabled. 8/8 correct at 589 estimated thinking tokens and 13.8 s total wall — efficient, no runaway loops on the reasoning channel.

**What it evaluates:** Scores the cost of reasoning: correctness, wall time, and estimated thinking tokens. All Qwen3-family models use a `[think]` channel; this measures how much budget it burns.

## Results

| Model |  Origin |  Score |  Est. thinking tokens |  Wall |  Note |
|---|---|--|--|--|--|--|
| **Qwen3.8-Flash-Next** (IQ3_XXS, 6B active) | China (AT pub.) | 8/8 | 589 | 13.8 s | 78.5 tok/s; 0.2 min

**Hardware:** 2× modded RTX 3080 20 GB — i7-8700K, 256 GiB, Ubuntu 24.04 — Strata 0.1.57 + DFlash2.

