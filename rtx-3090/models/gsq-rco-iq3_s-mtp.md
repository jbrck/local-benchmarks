# Qwen3.8-27B-GSQ-RCO-IQ3_S-mtp [kept]

**Role:** Smallest footprint, long context, vision.

## Results

| Test | Score |
|---|---|
| MATH-500 | 85.4% |
| HumanEval+ | 91.5% |
| IFEval prompt/instruction | 79.3% / 85.0% |
| GPQA-Diamond | 48.0% |
| Tool-eval-bench | 88 (★★★★) |
| Instruction v2 | 18/20 |
| Prose ELO | 1539 ‡ |
| Reasoning bench | not run |
| Refusal HARMFUL | not run |

**Status:** kept, footprint champion

## Timing

| Test | Total | Per item | Engine |
|---|---|---|---|
| MATH-500 (500 items) | *timing not captured* | *N/A* | llama.cpp GGUF |
| HumanEval+ (164 items) | **9min** | 3s | llama.cpp GGUF |
| GPQA-Diamond (198 items) | **19min** | 6s | llama.cpp GGUF |

## Notes

ISTA-DASLab's per-tensor mixed-precision quant at 3.5 bpw. The headline: **11.8 GB file, full 131K context at 14.9 GB VRAM** — same scores as the 19.8 GB Q5_K_M at 62% of the size. MATH-500 ties the 19.8 GB file (85.4 vs 86.8 — within noise), HumanEval+ matches it perfectly (91.5), IFEval is actually better (79.3/85.0 vs 77.3/83.5). 1.6x faster wall time due to less data through PCIe. The lab's "task-lossless" claim held on this box. Also carries the mmproj for vision — the only variant that can do multimodal.