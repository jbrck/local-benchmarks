# MATH-500 (Multi-step Math Reasoning)

**TL;DR:** 500 competition-style math problems (MATH hardest split), free-form, graded for exact correctness. 83.2% here — solid mid-pack for the Qwen3 class; the MoE's 6B-active size costs nothing on this leg.

**What it evaluates:** Multi-step algebra, combinatorics, probability, number theory. No tools, no code execution — pure chain-of-thought quality.

## Results

| Model |  Origin |  Accuracy |  Notes |
|---|---|--|--|--|
| **Qwen3.8-Flash-Next** (IQ3_XXS, 6B active) | China (AT pub.) | 83.2% (416/500) | thinking off; 46.4 min shared phase with HumanEval+

**Hardware:** 2× modded RTX 3080 20 GB — 2× Intel Xeon Gold 6138, 262 GiB, Ubuntu 24.04 — Strata 0.1.57 + DFlash2.

