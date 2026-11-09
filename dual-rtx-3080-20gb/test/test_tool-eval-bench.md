# Tool-eval-bench (Agentic Tool Use)

**TL;DR:** 69 agentic tool-use scenarios. 90/100 ★★★★★ with the safety gate passed — the clean-gate + strong-score combination only a few models on the 3090 card achieved. One weak category: Structured Output at 50%.

**What it evaluates:** 16 categories: tool selection, parameter precision, multi-step chains, restraint/refusal, error recovery, structured output, prompt-injection resistance. Multi-turn loops with mock tools.

## Results

| Model |  Origin |  Score |  Rating |  Safety warnings |
|---|---|--|--|--|--|
| **Qwen3.8-Flash-Next** (IQ3_XXS, 6B active) | China (AT pub.) | 90 | ★★★★★ | **none — gate passed**

**Hardware:** 2× modded RTX 3080 20 GB — i7-8700K, 256 GiB, Ubuntu 24.04 — Strata 0.1.57 + DFlash2.


Category detail for the 90: the only sub-100% category is **O Structured Output (50%)**; Tool Selection, Parameter Precision, Multi-Step Chains, Restraint & Refusal, Error Recovery, Localization, Structured Reasoning all 100%. Deployability 78, responsiveness 51, median turn 2.9 s, 566,857 tokens. Run 10.8 min at 133 tok/s.
