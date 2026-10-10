# bonsai2 (PrismML Ternary Bonsai 2 27B) [kept]

**TL;DR:** Tiny VRAM, 128K context. Qwen-family-adjacent scores (MATH 85.6, HE+ 89.0) in a small package.

**Role:** Tiny VRAM, 128K context — runs where nothing else fits.

## Results

| Test | Score |
|---|---|
| MATH-500 | 85.6% |
| HumanEval+ | 89.0% |
| IFEval prompt/instruction | 74.3% / 81.8% |
| GPQA-Diamond | 43.9% |
| Tool-eval-bench | 87 (★★★★) |
| Instruction v2 | 18/20 |
| Reasoning bench | not run |
| Refusal HARMFUL | not run |
| Context ceiling | 128K native, ~10 GB VRAM |

**Status:** kept: tiny VRAM, 128K context

## Timing

| Test | Total | Per item | Engine |
|---|---|---|---|
| HumanEval+ (164 items) | **9min** | 3s | llama.cpp GGUF |
| GPQA-Diamond (198 items) | **24min** | 7s | llama.cpp GGUF |

## Notes

Qwen3-QWQ-style ternary model from prism-ml (PTQ1_0 quant, 5.9 GB file). Runs at 128K context in only ~10 GB VRAM thanks to hybrid SSM architecture (qwen35 arch: SSM layers have no growing KV cache). Scores are solidly mid-pack — behind every qwen sibling on every axis, but at a fraction of the VRAM. The right model when memory is too tight for the full qwen family. Served via a PrismML-specific llama.cpp fork (ternary ggml type 143).