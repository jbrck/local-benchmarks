# LiveCodeBench (Competition Coding)

**TL;DR:** 100 competition coding problems, contamination-free window (post-2024-05). 62.0% — a genuinely complete run: all 100 generated, 99 scored, no budget truncation.

**What it evaluates:** Hidden tests on fresh problems; the modern pass@1 standard for local coding. The budget-truncated runs on the 3090 card (Bolt 54/100, Arete 75/100, Nanbeige 55/100) make a complete run the first thing to check.

## Results

| Model |  Origin |  Pass@1 |  Details |
|---|---|--|--|--|
| **Qwen3.8-Flash-Next** (IQ3_XXS, 6B active) | China (AT pub.) | 62.0% (62/100) | 99 scorable (1 no-code, 1 error); 55.6 min, 143.3 tok/s

**Hardware:** 2× modded RTX 3080 20 GB — 2× Intel Xeon Gold 6138, 262 GiB, Ubuntu 24.04 — Strata 0.1.57 + DFlash2.

