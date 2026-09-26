# crack2 (PQ2_0 abliterate) [experimental]

**Role:** Weight-abliterated Bonsai2 variant. Uncensored, but coding collapsed.

## Results

| Test | Score |
|---|---|
| MATH-500 | 83.6% |
| HumanEval+ | **76.2%** — severe coding collapse |
| IFEval prompt/instruction | 77.3% / 84.2% |
| GPQA-Diamond | 43.4% |
| Tool-eval-bench | 86 (★★★★) |
| Instruction v2 | **19/20** |
| Refusal (all tiers) | 0/30, 0/20, 0/20 — all empty output |

**Status:** kept (experimental)

## Notes

A PQ2_0 (7.2 GB) weight-abliterated variant of Bonsai2. The abliteration removed all safety refusal, but also destroyed coding ability — HumanEval+ at 76.2% is the second-worst score on this hardware. Output on refusal prompts was almost entirely empty strings (not refusal, just blank). IFEval and instruction-following (19/20) were surprisingly strong. Not suitable for agentic or coding use — purely experimental.