# qwen3.8-27b (GGUF Q5_K_M) [kept]

**TL;DR:** The baseline every model is judged against. Q5_K_M GGUF; strong everywhere (MATH 86.8, HE+ 91.5, tool 88), beaten only by specialized siblings.

**Origin:** Base weights — **China** (**Alibaba Qwen**) · Publisher — **Alibaba** (China). This is the origin point of the family — every "qwen3.8" derivative below inherits its Chinese provenance

**Role:** Baseline, best coding among the qwen siblings.

## Tested on

| Hardware | GPU | CPU | RAM | OS | Setup details |
|---|---|---|---|---|---|
| [RTX 3090 24 GB](../rtx-3090/README.md) | NVIDIA GeForce RTX 3090 24 GB (power-limited to 300 W) | Intel Core i7-8700K, 6 cores / 12 threads | 64 GB | Ubuntu 24.04 | [rtx-3090 README](../rtx-3090/README.md#system) |

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
