# nous-deepseek-v4-flash (remote) [baseline]

**TL;DR:** Remote baseline, not local. Tops every raw score (MATH 98.2, HE+ 93.9, GPQA 83.8, tool 93) — the datacenter bar local models chase.

**Role:** Remote baseline — shows what a frontier model costs vs local.

## Results

| Test | Score |
|---|---|
| MATH-500 | 98.2% |
| HumanEval+ | 93.9% |
| IFEval prompt/instruction | 86.7% / 90.9% |
| GPQA-Diamond | 83.8% |
| Tool-eval-bench | 93 (★★★★★) |
| Instruction v2 | not run |
| Reasoning bench | not run |
| Refusal HARMFUL | not run |

**Status:** remote baseline, not local

## Timing

| Test | Total | Per item | Engine |
|---|---|---|---|
| MATH-500 (500 items) | **1.2h** | 8s | LiteLLM (remote API) |
| HumanEval+ (164 items) | **30min** | 11s | LiteLLM (remote API) |
| GPQA-Diamond (198 items) | **37min** | 11s | LiteLLM (remote API) |

## Notes

The gulf between local and cloud. This model (served via API through a model proxy) crushes local on MATH (98.2 vs best local 89.2), GPQA (83.8 vs best local 50.5), and IFEval (86.7/90.9 vs best local 80.6/85.8). On coding, the gap is much smaller: 93.9 vs best local 94.5 (ornith). The local models are surprisingly close on code generation despite being ~2-4 years behind on reasoning and knowledge breadth.