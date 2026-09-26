# nous-deepseek-v4-flash (remote) [baseline]

**Role:** Remote baseline — shows what a frontier model costs vs local.

## Results

| Test | Score |
|---|---|
| MATH-500 | 98.2% |
| HumanEval+ | 93.9% |
| IFEval prompt/instruction | 86.7% / 90.9% |
| GPQA-Diamond | 83.8% |
| Tool-eval-bench | 93 (★★★★★) |

**Status:** remote baseline, not local

## Notes

The gulf between local and cloud. This model (served via API through a model proxy) crushes local on MATH (98.2 vs best local 89.2), GPQA (83.8 vs best local 50.5), and IFEval (86.7/90.9 vs best local 80.6/85.8). On coding, the gap is much smaller: 93.9 vs best local 94.5 (ornith). The local models are surprisingly close on code generation despite being ~2-4 years behind on reasoning and knowledge breadth.