# Gemma-4-26B

**TL;DR:** Rejected. Far below the field on every leg (MATH 48.8%, HE+ 56.1%, GPQA 15.0%). Partial battery only.

| Property | Detail |
|---|---|
| Source | Google DeepMind |
| Architecture | Dense 26B |
| Format | GGUF (Q4_K_M, ~15 GB) |
| Engine | llama.cpp (port 11434) |
| Context | 8,192 tokens (tested) |
| VRAM | ~16 GB |

## Partial battery

| Test | Score | Notes |
|---|---|---|
| MATH-500 | **48.8%** | Far below 27B qwen family (86-94%). The 26B dense architecture is outperformed by compressed 27B MoEs. |
| HumanEval+ | **56.1%** | Weak for its size. |
| GPQA-Diamond | **15.0%** | Near chance on graduate science MCQs. |

Not competitive with any qwen3.8-27B variant. No further testing justified. Marked as rejected.