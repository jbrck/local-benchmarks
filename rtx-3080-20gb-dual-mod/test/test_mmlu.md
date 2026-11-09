# MMLU (Multi-Subject Knowledge)

**TL;DR:** 500-item multi-subject subset (57 subjects). 90.8% — the best knowledge number recorded on this box; the MoE's strongest leg.

**What it evaluates:** Knowledge retention and recall across STEM, humanities, and social science. Generative answering, 500-item subset, 4-way multiple choice.

## Results

| Model |  Origin |  Accuracy |  Notes |
|---|---|--|--|--|
| **Qwen3.8-Flash-Next** (IQ3_XXS, 6B active) | China (AT pub.) | **91.0%** (455/500)[†](#fn-thinkon) | thinking-on rerun; thinking-off baseline 90.8% (454/500) — flat, recall does not benefit from reasoning

<a id="fn-thinkon">†</a> Thinking-on rerun 2026-11-09; battery default is thinking-off.

**Hardware:** 2× modded RTX 3080 20 GB — 2× Intel Xeon Gold 6138, 262 GiB, Ubuntu 24.04 — Strata 0.1.57 + DFlash2.

