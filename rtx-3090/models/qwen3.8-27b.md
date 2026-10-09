# qwen3.8-27b (GGUF Q5_K_M) [kept]

**Role:** Baseline, best coding among the qwen siblings.

## Results

| Test | Score |
|---|---|
| MATH-500 | 86.8% |
| HumanEval+ | 91.5% |
| IFEval prompt/instruction | 77.3% / 83.5% |
| GPQA-Diamond | 46.0% |
| Tool-eval-bench | 88 (★★★★) |
| Instruction v2 | 18/20 |
| Reasoning bench | baseline |

**Status:** kept

## Timing

| Test | Total | Per item | Engine |
|---|---|---|---|
| MATH-500 (500 items) | *timing not captured* | *N/A* | llama.cpp GGUF |
| HumanEval+ (164 items) | **18min** | 7s | llama.cpp GGUF |
| GPQA-Diamond (198 items) | **52min** | 16s | llama.cpp GGUF |
| IFEval (541 prompts) | **1h45m** | ~12s | llama.cpp GGUF |

## Notes

The Qwen3.8-27B family owns this GPU. This is the vanilla Qwen3.8-27B at Q5_K_M quantization — the baseline every other model was compared against. Strong across the board, slightly edged out by specialized variants on individual tests.

---

<a id="fn-dagger">‡</a> `--no-think` mode.
