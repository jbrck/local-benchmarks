# exl3-qwen3.8-27b (3.5bpw) [engine comparison]

**Role:** EXL3 serving variant — same weights as qwen3.8-27b, different engine.

## Results

| Test | Score |
|---|---|
| MATH-500 | 85.8% |
| HumanEval+ | 87.8% (4096-cap rerun; 74.4% at 1024-cap — artifact) |
| IFEval prompt/instruction | 80.6% / 85.8% |
| GPQA-Diamond | 46.5% |
| Tool-eval-bench | 91 (★★★★) |

**Status:** engine comparison, same weights as qwen3.8-27b GGUF

## Notes

Same Qwen3.8-27B weights, served via exllamav3 at 3.5 bpw instead of llama.cpp GGUF. Always reasons (no off switch), which makes it slower and more token-hungry on coding tasks — HumanEval+ needed a 4096-token cap to produce code at all, and still trailed the GGUF variant by ~4-5 points. Where it wins: IFEval (best local prompt/instruction scores 80.6/85.8) and raw decode throughput (~4x GGUF at 105 vs 27 tok/s). Also holds 196K context at 21.4 GB VRAM (vs GGUF 131K at 18-19 GB). The throughput advantage is real, but reasoning can't be disabled, so it's best for long-context or high-volume work where reasoning-on is acceptable.