# HumanEval+ (Python Code Generation)

**TL;DR:** 164 Python problems graded against hidden test suites (EvalPlus). 82.9% — comparable to the Qwen3 27B-class row on the 3090 card; code emission is efficient at ~95 tokens/item.

**What it evaluates:** Function-level code generation; hidden test cases catch subtle correctness. Measures raw code-writing, not tool use.

## Results

| Model |  Origin |  Pass@1 |  Notes |
|---|---|--|--|--|
| **Qwen3.8-Flash-Next** (IQ3_XXS, 6B active) | China (AT pub.) | 82.9% (136/164) | thinking off; part of the shared math+code 46.4 min phase

**Hardware:** 2× modded RTX 3080 20 GB — i7-8700K, 256 GiB, Ubuntu 24.04 — Strata 0.1.57 + DFlash2.

