# GPQA-Diamond (Graduate Science Knowledge)

**TL;DR:** 198 graduate-level science multiple-choice questions (biology, chemistry, physics). 50.0% — mid-pack for the Qwen3 base class; the 6B-active MoE's weak leg.

**What it evaluates:** Wikipedia-level + research-level science reasoning. Diamond split only; random chance is 25% (4-way).

## Results

| Model |  Origin |  Accuracy |  Notes |
|---|---|--|--|--|
| **Qwen3.8-Flash-Next** (IQ3_XXS, 6B active) | China (AT pub.) | 50.0% (99/198) | thinking off; 7.8 min, 145.6 tok/s

**Hardware:** 2× modded RTX 3080 20 GB — 2× Intel Xeon Gold 6138, 262 GiB, Ubuntu 24.04 — Strata 0.1.57 + DFlash2.

