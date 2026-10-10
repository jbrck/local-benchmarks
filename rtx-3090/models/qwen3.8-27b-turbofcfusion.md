# Qwen3.8-27B-TurboFCFusion ("turbo-fable") (GGUF Q4_K_M)

**TL;DR:** Strong coder (86.0% HE+), mediocre elsewhere (MATH 81.0, GPQA 35.9). 'Turbo' speed claims did not hold. Token-hungry.

**Status:** tested | **Decision:** strong coder, mediocre elsewhere. The "TURBO / CODER MAX" claims hold for code only; speed is a lie.

| Spec | Value |
|---|---|
| Size | 27B |
| Quant | Q4_K_M (GGUF) |
| Engine | llama.cpp |
| Speed | ~31 tok/s (measured, 300 W power cap) |
| VRAM | ~18 GB at 8K ctx (24 GB card) |
| Architecture | qwen3.8 (+ reasoning) |
| Context | 8K tested (256K theoretical) |
| Chat format | reasoning_content split — heavy mandatory thinking before content |

## Results

This model thinks before answering — every request burns tokens in `reasoning_content` first. It needs a ~2-3x token budget to produce content at all, so all scores below are at or near that elevated budget.

| Test | Score | vs 27B field | Notes |
|---|---|---|---|
| MATH-500 | **81.0%** (405/500) | below qwen3.8-27b (86.8%), turbo-fable underperforms | Consistent 80-82% throughout, no tail drift |
| HumanEval+ | **86.0%** (141/164) | 2nd best (behind ornith-1.5-35b's 94.5%) | "CODER MAX" claim holds — genuinely good |
| GPQA-Diamond | **35.9%** (71/198) | low end of 27B field (35-40%) | Barely above random (25%) |
| Instruction v2 | **95.0%** (19/20) | best of the 27B class | Strong constraint following |

## Analysis

### Strengths
- **HumanEval+ 86.0%** is the best 27B coding score I've seen this generation (behind only ornith-1.5-35b's 94.5% at a larger 35B). The "CODER MAX" in the name is real for code.
- **Instruction v2 95.0%** (19/20) — best mechanical constraint-following of the 27B Qwen family. Only missed max_words.
- Prose quality matches the baseline qwen3.8-27b when given enough tokens (10/10 draws head-to-head).

### Weaknesses
- **MATH 81.0%** is below the base model's 86.8% — the "fusion" variant loses math ability.
- **GPQA 35.9%** is random-level, on par with the weakest 27B variants.
- **Slow.** 31 tok/s vs 43 for the base Qwen3.8-27b at Q4_K_M — the reasoning overhead makes every task 3-5x slower wall-clock. MATH-500 took 2.5+ hours vs ~30 min for the baseline.
- **Token-hungry.** Needs 3000-token budgets to produce prose; 1500-token runs come back empty.
- The 24 GB card's 300 W power cap trims all tok/s by ~15-20% (see README note).

### Notes
- All battery processes used `setsid` + `PYTHONPATH=` for Hermes lifecycle isolation.
- Bench scripts needed resume-arg support (`--math-start-index`, `--math-known-correct`) after partial runs — added to `bench_full.py`.
- Bottom line: a code specialist with a "TURBO" name that isn't, and prose that only shows up if you pay 2-3x the token bill.

## Timing

| Phase | Wall time |
|---|---|
| MATH-500 | ~2.5h+ (partial run resumed twice; ~29.8s/problem avg) |
| HumanEval+ | ~55 min (20.1s/problem avg) |
| GPQA-Diamond | ~1h 40m |
| Instruction v2 | ~10 min |
| **Total** | **~5.5h** |