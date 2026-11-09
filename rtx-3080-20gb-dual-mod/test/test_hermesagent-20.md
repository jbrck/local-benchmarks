# HermesAgent-20 (Agentic Harness Behavior)

**TL;DR:** 20 scenarios driving a real pinned agent runtime in Docker, graded on artifacts (memory, files, cron, approvals). 13/20 pass@1, 14/20 best-of-3 (70%), zero runaway loops, 469,498 tokens, 13.2 min.

**What it evaluates:** Whether the model runs a loop to a correct end state — writes memory, patches a skill, schedules a job, walks an approval, delegates — not whether it answers well. Artifact-graded: memory state, files on disk, cron entries, delivery logs.

## Results

| Model |  Origin |  Score |  Scenarios |  Runaway |  Tokens |  Wall |  Notes |
|---|---|---|---|---|---|---|---|
| **Qwen3.8-Flash-Next** (IQ3_XXS, 6B active) | China (AT pub.) | 13/20 (65%) pass@1; 14/20 (70%) pass@k[‡](#fn-dagger) | 20 | 0 | 469,498 | 13.2 min | thinking forced off; p50 turn 21.6 s; 1 flaky credited, 7 retried

**Hardware:** 2× modded RTX 3080 20 GB — 2× Intel Xeon Gold 6138, 262 GiB, Ubuntu 24.04 — Strata 0.1.57 + DFlash2.


<a id="fn-dagger">‡</a> Forced `--no-thinking` (thinking_mode `force-off`) though the pack defaults to thinking-enabled — matches the rest of this battery. 451,024 prompt / 18,474 completion tokens; sandbox agent pinned by the runner (upstream commit recorded in the run JSON).
