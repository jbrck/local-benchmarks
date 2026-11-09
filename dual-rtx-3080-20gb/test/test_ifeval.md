# IFEval (Instruction Following)

**TL;DR:** 541 prompts with mechanical constraints. 80.4% prompt / 85.8% instruction accuracy — fine for this class, consistent with the 3090 card's Qwen3-family results.

**What it evaluates:** Prompt-instruction scores measure whether the model follows formatting constraints exactly (length, format, keyword) when asked to write.

## Results

| Model |  Origin |  Prompt Accuracy |  Instruction Accuracy |  Notes |
|---|---|--|--|--|--|
| **Qwen3.8-Flash-Next** (IQ3_XXS, 6B active) | China (AT pub.) | 80.4% [◊](#fn-ifeval) | 85.8% [◊](#fn-ifeval) | 435/541, 716/834; 32.3 min, 105.4 tok/s

**Hardware:** 2× modded RTX 3080 20 GB — 2× Intel Xeon Gold 6138, 262 GiB, Ubuntu 24.04 — Strata 0.1.57 + DFlash2.


<a id="fn-ifeval">◊</a> Read from the battery driver log — the harness wrote no separate IFEval artifact this run.
