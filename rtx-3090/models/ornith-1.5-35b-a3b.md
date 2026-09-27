# ornith-1.5-35b-a3b (q4_k_s) [kept]

**Role:** Best coding, but heavy and narrow.

## Results

| Test | Score |
|---|---|
| MATH-500 | 85.2% |
| HumanEval+ | **94.5%** — best local |
| IFEval prompt/instruction | 75.0% / 82.7% |
| GPQA-Diamond | **34.3%** — worst local |
| Tool-eval-bench | 88 (★★★★) |
| Instruction v2 | **19/20** |
| Refusal HARMFUL | 19/20 (95%) — safe profile |
| Prose ELO | not run |
| Reasoning bench | not run |

**Status:** kept, best coding

## Timing

| Test | Total | Per item | Engine |
|---|---|---|---|
| MATH-500 (500 items) | *timing not captured* | *N/A* | llama.cpp GGUF |
| HumanEval+ (164 items) | **11min** | 4s | llama.cpp GGUF |
| GPQA-Diamond (198 items) | **21min** | 6s | llama.cpp GGUF |

## Notes

A 35B MoE model (21 GB GGUF, 0.7 GB free at load) — the only non-27B model that survived screening. Best local coding by a wide margin (94.5% HumanEval+), and best instruction-following alongside swift (19/20). Safe refusal profile (95% harmful, 0% over-refusal). But GPQA-Diamond at 34.3% is a knowledge gap that makes it unreliable for general reasoning tasks. Only fits 64K context on the 3090 (21 GB file leaves no room for deeper KV). Runs at the VRAM ceiling — one more GB and it wouldn't fit.