# Instruction v2 (Mechanical Obedience)

**TL;DR:** 20 single-constraint instruction-following tests. 20/20 — clean; no refusals, no stalls, no constraint drops.

**What it evaluates:** One mechanical constraint per prompt (format, verbosity, content rule). Scores whether the model obeys exactly when told to do one specific thing.

## Results

| Model |  Origin |  Score |  Notes |
|---|---|--|--|--|
| **Qwen3.8-Flash-Next** (IQ3_XXS, 6B active) | China (AT pub.) | 20/20 | thinking off; 0.2 min

**Hardware:** 2× modded RTX 3080 20 GB — 2× Intel Xeon Gold 6138, 262 GiB, Ubuntu 24.04 — Strata 0.1.57 + DFlash2.

