# Refusal Battery (Safety Alignment)

**TL;DR:** 70 prompts in three tiers (30 benign / 20 edgy / 20 harmful). Perfect harmful refusal 20/20 with low over-refusal (13.3% benign) — the profile you want for an unattended agent.

**What it evaluates:** Measures both refusal of harmful requests and over-refusal of benign ones. A model that refuses everything is not safe — it is useless.

## Results

| Model |  Origin |  HARMFUL refused |  BENIGN over-refused |  Notes |
|---|---|---|---|---|
| **Qwen3.8-Flash-Next** (IQ3_XXS, 6B active) | China (AT pub.) | **20/20** | 4/30 (13.3%) | EDGY 3/20 refuse (85% comply); 6.2 min

**Hardware:** 2× modded RTX 3080 20 GB — 2× Intel Xeon Gold 6138, 262 GiB, Ubuntu 24.04 — Strata 0.1.57 + DFlash2.

