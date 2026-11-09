# Qwen3.8-Flash-Next (GSQ-RCO IQ3_XXS)

**TL;DR:** Alibaba Qwen's 125B-parameter MoE (6B active) at IQ3_XXS, served at 64K context by Strata + DFlash2 on dual modded 3080s. First full battery on this box, and the reasoning legs get a lot better with reasoning on: **GPQA-D 78.3%, MATH-500 91.0%, MMLU 91.0%** on [thinking-on reruns](#fn-thinkon) (vs 50.0 / 83.2 / 90.8% thinking-off). Add a **complete LiveCodeBench 62.0%**, tool-eval **90/100 ★★★★★ with a passed safety gate**, perfect harmful refusal (20/20) with low over-refusal, HermesAgent-20 13/20 pass@1 (70% best-of-3) with zero runaways, and the weak spot narrows to **IFBench ~30% strict**. Speed is the headline: **~100–165 tok/s** for a 125B MoE — both battery passes (13 legs off + 3 thinking-on legs) fit in one night.

**Origin:** Base weights — **China** (Alibaba Qwen3.8-Flash-Next) · Publisher of this file — **GSQ-RCO by ISTA-DASLab** (Austria)

**Spec:** IQ3_XXS ~75.8 GB pack · 125B MoE / 6B active · 64K ctx (int8 KV) · Strata 0.1.57 + DFlash2 · thinking-off default

## Results

| Test | Score |
|---|---|
| MATH-500 | **91.0%** (455/500)[†](#fn-thinkon) — thinking-on rerun; off was 83.2% |
| HumanEval+ | 82.9% (136/164) |
| IFEval prompt/instruction | 80.4% / 85.8% (435/541, 716/834) [◊](#fn-ifeval) |
| GPQA-Diamond | **78.3%** (155/198)[†](#fn-thinkon) — thinking-on rerun; off was 50.0% |
| Tool-eval-bench | 90 (★★★★★) — safety gate **passed**, no warnings; only weak category Structured Output (50%) |
| Instruction v2 | 20/20 |
| Refusal HARMFUL | **20/20 (100%)** |
| Refusal BENIGN (over-refusal) | 4/30 (13.3%) |
| MMLU | **91.0%** (455/500)[†](#fn-thinkon) — thinking-on rerun; off was 90.8% |
| IFBench strict/loose | 28.0 / 30.7% prompt · 28.8 / 32.6% instruction |
| Reasoning bench | 8/8 (589 est. thinking tokens, 13.8 s) |
| HermesAgent-20 | 13/20 (65%) pass@1 · 14/20 (70%) pass@k [‡](#fn-dagger) |
| LiveCodeBench | 62.0% (62/100) — full run, no truncation |

**Status:** tested — strong all-round agentic profile on this box; clean safety gate + perfect harmful refusal; reasoning legs need thinking on to show it (IFBench is the genuine weak leg)

## Timing

| Leg | Total | Per item | Engine |
|---|---|---|---|
| MATH-500 + HumanEval+ (664 items) | 46.4 min | 4.2s | Strata + DFlash2 |
| GPQA-Diamond (198 items) | 7.8 min | 2.4s | Strata + DFlash2 |
| Instruction v2 (20 items) | 0.2 min | 0.6s | Strata + DFlash2 |
| Refusal (70 prompts) | 6.2 min | 5.3s | Strata + DFlash2 |
| Reasoning bench (8 tasks) | 0.2 min | 1.5s | Strata + DFlash2 |
| IFEval (541 prompts) | 32.3 min | 3.6s | Strata + DFlash2 |
| IFBench (300 items) | 16.8 min | 3.4s | Strata + DFlash2 |
| MMLU (500 items) | 44.5 min | 5.3s | Strata + DFlash2 |
| LiveCodeBench (100 problems) | 55.6 min | 33.4s | Strata + DFlash2 |
| Tool-eval-bench (69 scenarios) | 10.8 min | 9.4s | Strata + DFlash2 |
| HermesAgent-20 (20 scenarios) | 13.2 min | 39.6s | Strata + DFlash2 |

### Thinking-on rerun legs (2026-11-09)

| Leg | Total | Avg tokens/item | Engine |
|---|---|---|---|
| MATH-500 (500 items) | 62.2 min | 922 | Strata + DFlash2, thinking on |
| GPQA-Diamond (198 items) | 60.1 min | 2,181 | Strata + DFlash2, thinking on |
| MMLU (500 items) | 47.4 min | ~1,400 | Strata + DFlash2, thinking on |

## Notes

The profile splits cleanly into two verdicts. **Instruction and agentic legs are strong.** MMLU 91.0% (generative, 500-item subset, thinking-on rerun) is the best knowledge number recorded on this box; LiveCodeBench finished all 100 problems (62 passed, 1 no-code, 1 error) with zero truncation; tool-eval scored 90 with the safety gate clean at the category level — every category 100% except **O Structured Output (50%)** and a couple of partials; harmful refusal is perfect (20/20) and benign over-refusal is low (13.3%), the profile you want for an unattended agent. HermesAgent-20 at 13/20 pass@1 / 14/20 best-of-3 used **469,498 tokens** (451,024 prompt / 18,474 completion) with zero runaway loops and a 21.6 s p50 per agent turn — lean for a 20-scenario harness run.

**Reasoning is strong — when it is allowed to reason.** The thinking-on rerun is the headline of this page: MATH-500 **83.2% → 91.0%** (39 more correct), GPQA-D **50.0% → 78.3%** (56 more correct). The model is RL-trained to reason and the harness default caps cost it real points on the first pass. GPQA reasoning runs deep — 2,181 avg tokens/item, some items past 10K reasoning tokens before a 9-token answer. MMLU barely moved (90.8% → 91.0%, +1 item): knowledge recall does not benefit from reasoning, which is the expected shape for this family. HumanEval+ 82.9% and IFBench ~30% strict are the two legs thinking cannot fix; IFBench's 28.0/30.7% strict/loose prompt accuracy is a formatting-strictness gap, not an instruction-following gap (IFEval on the same axis is 80.4/85.8%).

**Speed is the structural win.** A 125B MoE with 6B active on dual 20 GB cards ran every leg at 100–165 tok/s, which is why a 13-leg battery plus three thinking-on reruns fit in one night. The DFlash2 speculative decode is doing real work here; this is the fastest large-MoE throughput this repo has recorded on two 3080-class cards. Thinking-on roughly doubles per-item time on reasoning legs (MATH ~4.2 → 7.5 s/item) but stays comfortably faster than a dense 27B on the 3090 card.

The battery ran thinking-off end to end. HermesAgent-20's pack defaults to thinking-on; it was forced off ([‡](#fn-dagger)) so the leg stays comparable with the rest of the battery.

---

<a id="fn-dagger">‡</a> HermesAgent-20 forced `--no-thinking` (`thinking_mode: force-off`), different from the pack's default. 13/20 pass@1, 14/20 best-of-3 (7 scenarios retried, 1 flaky credited), 0 runaway, 469,498 tokens, sandbox agent pinned by the runner (upstream commit recorded in the run JSON).
<a id="fn-ifeval">◊</a> IFEval 80.4 / 85.8% (435/541, 716/834) read from the battery driver log; no separate IFEval artifact was written this run.

<a id="fn-thinkon">†</a> Thinking-on rerun (2026-11-09, `enable_thinking`). The battery default is thinking-off; these three legs reran with thinking enabled for comparability with the [rtx-3090](the rtx-3090 section) rows that used thinking on (Mirai S, Nanbeige MMLU). Deltas: MATH-500 **83.2% → 91.0%** (+7.8, 416→455 correct), GPQA-D **50.0% → 78.3%** (+28.3, 99→155), MMLU **90.8% → 91.0%** (+0.2, 454→455 — knowledge recall does not benefit from reasoning). Token caps raised on the rerun for reasoning room: MATH 2048→4096, GPQA 1024→4096 (MMLU already 4096).