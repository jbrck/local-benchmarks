# exl3-qwen3.8-27b (3.5bpw) [engine comparison]

**TL;DR:** EXL3 engine comparison of the same qwen3.8-27b weights. Scores reflect forced-reasoning mode — not directly comparable to GGUF runs.

**Role:** EXL3 serving variant — same weights as qwen3.8-27b, different engine.

## Results

| Test | Score |
|---|---|
| MATH-500 | 85.8% § |
| HumanEval+ | 87.8% § (4096-cap rerun; 74.4% at 1024-cap — artifact) |
| IFEval prompt/instruction | 80.6% / 85.8% § |
| GPQA-Diamond | 46.5% [§](#fn-section) |
| Tool-eval-bench | 91 § (★★★★) |
| Instruction v2 | not run |
| Reasoning bench | — [§](#fn-section) |
| Refusal HARMFUL | not run |

**Status:** engine comparison, same weights as qwen3.8-27b GGUF

## Timing

| Test | Total | Per item | Engine |
|---|---|---|---|
| MATH-500 (500 items) | **57min** | 7s | EXL3 3.5bpw |
| HumanEval+ (164 items) | **18min** | 6s | EXL3 3.5bpw |
| GPQA-Diamond (198 items) | **35min** | 11s | EXL3 3.5bpw |

## Notes

Same Qwen3.8-27B weights, served via exllamav3 at 3.5 bpw instead of llama.cpp GGUF. Always reasons (no off switch), which makes it slower and more token-hungry on coding tasks — HumanEval+ needed a 4096-token cap to produce code at all, and still trailed the GGUF variant by ~4-5 points. Where it wins: IFEval (best local prompt/instruction scores 80.6/85.8) and raw decode throughput (~4x GGUF at 105 vs 27 tok/s). Also holds 196K context at 21.4 GB VRAM (vs GGUF 131K at 18-19 GB). The throughput advantage is real, but reasoning can't be disabled, so it's best for long-context or high-volume work where reasoning-on is acceptable.

---

<a id="fn-section">§</a> Reasoning forced — model architecture cannot disable thinking.
