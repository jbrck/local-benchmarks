# MMLU

**TL;DR:** 500-question sample of the 57-subject exam. Mirai S Qwen3.8-27B scored 87.0% (435/500) — 2.8 points under the author's 89.8% claim, on a different sample. Strong on math, CS, and elementary material; bleeds on medicine, law, and moral reasoning.

**What it evaluates:** The Massive Multitask Language Understanding benchmark — 14,042 multiple-choice questions across 57 subjects (STEM, humanities, social sciences, professional domains). This run samples 500 items. It measures retained knowledge, not reasoning depth: the model either knows the fact or it doesn't. Cheap to run, useful as a knowledge floor.

**Reading scores:** 85%+ is a healthy 27B. Below 80% suggests knowledge gaps or a parsing problem. This is not a hard reasoning test — a model can score high here and still fail MATH-500, or vice versa.

## Results

| Model | [Origin](../README.md#results-summary) | Accuracy | Notes |
|---|---|---|---|
| **Mirai S Qwen3.8-27B** (2.4 bpw trellis) | China | **87.0%** (435/500) | thinking on; author claims 89.8% — not reproduced on this sample |

## Subject breakdown

Sampled subjects with 5+ items, weakest first.

| Subject | Score | | Subject | Score |
|---|---|---|---|---|
| virology | 5/9 = 56% | | philosophy | 8/9 = 89% |
| high_school_physics | 4/6 = 67% | | prehistory | 11/12 = 92% |
| high_school_chemistry | 5/7 = 71% | | high_school_microeconomics | 11/12 = 92% |
| moral_disputes | 8/11 = 73% | | nutrition | 12/13 = 92% |
| professional_law | 36/49 = 73% | | high_school_macroeconomics | 16/17 = 94% |
| moral_scenarios | 29/39 = 74% | | high_school_psychology | 17/18 = 94% |
| human_aging | 7/9 = 78% | | miscellaneous | 29/30 = 97% |
| high_school_world_history | 8/10 = 80% | | elementary_mathematics | 16/16 = 100% |
| professional_psychology | 22/27 = 81% | | high_school_geography | 16/16 = 100% |
| marketing | 11/13 = 85% | | professional_accounting | 10/10 = 100% |
| clinical_knowledge | 12/14 = 86% | | high_school_statistics | 8/8 = 100% |

The pattern: precise, rule-based subjects (accounting, statistics, elementary math) score 100%. Subjects needing nuance or specialist recall (law, moral scenarios, virology) drag. Six subjects scored 100%, and the top and bottom of the range span 56% to 100% — the model has real breadth and real holes.

## Speed

| Metric | Value |
|---|---|
| Items | 500 |
| Correct | 435 (87.0%) |
| Wall clock | 5,016.6 s (83.6 min) |
| Completion tokens | 356,470 |
| Throughput | 71.1 tok/s |

## Scoring

Generative mode — the model answers in prose, the harness extracts the chosen letter (A/B/C/D) with a case-insensitive pattern match and compares against the gold key. No logprob shortcut. Wrong or missing letter counts as incorrect. Zero errors across the 500 items.

**Harness:** `bench_mmlu.py` (direct runner, `--selftest` proves the parser with unit tests before any scored run).

**Hardware:** RTX 3090 24 GB, llama.cpp (Mirai S-ada fork), q8_0 KV, MTP speculative decoding, 300 W power cap.
