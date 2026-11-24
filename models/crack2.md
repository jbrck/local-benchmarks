# crack2 (PQ2_0 abliterate) [experimental]

**TL;DR:** Weight-abliterated variant. Keeps tool-eval 86 and Instr 19/20, but HumanEval+ collapsed to 76.2%. Experimental.

**Origin:** Base weights — **China** (Bonsai2 ← Alibaba Qwen3.8-27B) · Publisher — community weight-abliterate, publisher country not established

**Role:** Weight-abliterated Bonsai2 variant. Uncensored, but coding collapsed.

## Tested on

| Hardware | GPU | CPU | RAM | OS | Setup details |
|---|---|---|---|---|---|
| [RTX 3090 24 GB](../rtx-3090/README.md) | NVIDIA GeForce RTX 3090 24 GB (power-limited to 300 W) | Intel Core i7-8700K, 6 cores / 12 threads | 64 GB | Ubuntu 24.04 | [rtx-3090 README](../rtx-3090/README.md#system) |

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
| Reasoning bench | not run |

**Status:** kept (experimental)

## Timing

| Test | Total | Per item | Engine |
|---|---|---|---|
| MATH-500 (500 items) | *timing not captured* | *N/A* | llama.cpp GGUF |
| HumanEval+ (164 items) | **6min** | 2s | llama.cpp GGUF |
| GPQA-Diamond (198 items) | **23min** | 7s | llama.cpp GGUF |

## Notes

A PQ2_0 (7.2 GB) weight-abliterated variant of Bonsai2. The abliteration removed all safety refusal, but also destroyed coding ability — HumanEval+ at 76.2% is the second-worst score on this hardware. Output on refusal prompts was almost entirely empty strings (not refusal, just blank). IFEval and instruction-following (19/20) were surprisingly strong. Not suitable for agentic or coding use — purely experimental.