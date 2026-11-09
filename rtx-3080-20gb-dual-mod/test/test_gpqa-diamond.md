# GPQA-Diamond (Graduate Science Knowledge)

**TL;DR:** 198 graduate-level science multiple-choice questions (biology, chemistry, physics). 50.0% — mid-pack for the Qwen3 base class; the 6B-active MoE's weak leg.

**What it evaluates:** Wikipedia-level + research-level science reasoning. Diamond split only; random chance is 25% (4-way).

## Results

| Model |  Origin |  Accuracy |  Notes |
|---|---|---|---|
| **Qwen3.8-Flash-Next** (IQ3_XXS, 6B active) | China (AT pub.) | **78.3%** (155/198)[†](#fn-thinkon) | thinking-on rerun (1h00m, avg 2,181 tok/item — some items past 10K reasoning); thinking-off baseline 50.0% (99/198)

<a id="fn-thinkon">†</a> Thinking-on rerun 2026-11-09; battery default is thinking-off. Token cap 4096 for reasoning room.

**Hardware:** 2× modded RTX 3080 20 GB — 2× Intel Xeon Gold 6138, 262 GiB, Ubuntu 24.04 — Strata 0.1.57 + DFlash2.

