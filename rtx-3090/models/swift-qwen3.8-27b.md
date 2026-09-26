# swift-qwen3.8-27b [kept — default]

**Role:** Default — safest safety profile, best math, best GPQA.

## Results

| Test | Score |
|---|---|
| MATH-500 | **89.2%** |
| HumanEval+ | 87.8% |
| IFEval prompt/instruction | 79.5% / 85.1% |
| GPQA-Diamond | **50.5%** |
| Tool-eval-bench | 89 (★★★★) |
| Instruction v2 | **19/20** |
| Prose ELO | 1479 |
| Reasoning bench (vs baseline) | 1.41x speedup, 21.7% fewer thinking tokens |
| Refusal HARMFUL | **20/20 (100%)** |
| Refusal BENIGN (over-refusal) | 5/30 (17%) |

**Status:** kept, default

## Notes

The Swiss Army knife of the local stack. Best local math score (89.2%), best local GPQA (50.5% — the only model above 50%), perfect harmful refusal (20/20), and best instruction-following (19/20). Its reasoning is 1.41x faster than vanilla qwen with 21.7% fewer thinking tokens (its marketing claimed 1.95x/58.3% — direction true, overstated). The tradeoff: coding is a notch behind (87.8% vs 91.5%) and prose is mid-pack (1479). Slight over-refusal on benign prompts (17%). Decode is ~22% slower than GSQ-RCO at equal per-token quality — an engine characteristic, not a model weakness.