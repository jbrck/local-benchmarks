# Mirai S Qwen3.8-27B

| Property | Detail |
|---|---|
| Source | [Mirai S](https://x.com/superalesha/status/2107896192591945829) (tweet) |
| Architecture | Qwen3.8-27B MoE (3.8B active / 27B total) |
| Format | 2.4 bpw trellis codec (GGUF, ~11 GB) |
| Engine | Mirai S-ada (llama.cpp fork) |
| Server | Port 8082, MTP speculative decoding, q8_0 KV |
| Context | **262,144** tokens (full theoretical max), VRAM ceiling ~384K |
| Speed | 54 tok/s (decode, all context sizes), 70 tok/s on code |
| VRAM | 19.1 GB (model + 262K q8_0 KV), ~5 GB free |

## Battery results

| Test | Score | Notes |
|---|---|---|
| MATH-500 | **94.2%** | Best local; +3.4 over OrcaSAQ-2-27B |
| HumanEval+ | **95.1%** | Best local; beats DeepSeek V4 Flash (93.9%) |
| GPQA-Diamond | **69.7%** | Best local; +13.1 over Muse Glimmer 30B (56.6%) |
| Instr v2 | **20/20 (100%)** | First perfect score on this test |
| IFEval | 78.9 / 85.1% | 3rd local; ties swift's instruction score. 49 min, 214k tokens |
| Tool-eval | **93/100** | Ties DeepSeek V4 Flash remote baseline; 28/30 points |
| Refusal | 20/20 harmful | 17% benign over-refusal, 10% edgy |
| Reasoning | 8/8, 126s | Correctness saturated; no thinking-token edge |
| Prose ELO | **1451** | Below qwen3.8-27b baseline (1704). Compression cost shows in writing |

## Context ceiling

All sizes tested with the server binary, full speculative decoding, and q8_0 KV cache. Decode speed is flat across all tested ranges (~54 tok/s).

| Context | VRAM | Free | Loads? |
|---|---|---|---|
| 131,072 | 14,961 MB | 9,039 MB | ✅ |
| 192,000 | 17,103 MB | 6,897 MB | ✅ |
| 240,000 | 18,795 MB | 5,205 MB | ✅ |
| 262,144 (rated max) | 19,569 MB | 4,431 MB | ✅ |
| 288,000 | 20,479 MB | 3,521 MB | ✅ |
| 300,000 | 20,901 MB | 3,099 MB | ✅ |
| 320,000 | 21,605 MB | 2,395 MB | ✅ |
| 350,000 | 22,667 MB | 1,333 MB | ✅ |
| 384,000 (VRAM wall) | 23,855 MB | 145 MB | ✅ |

The model loads at the manufacturer's rated 262K with ~4.4 GB headroom. Position embeddings degrade past 262K (extrapolation territory) but the card can physically hold up to ~385K tokens at q8_0. Swap to q4_0 KV and the ceiling doubles.

## Verdict

Best model tested on this machine — #1 on MATH-500, HumanEval+, GPQA-Diamond, and Instr v2. The 2.4 bpw trellis compression preserved reasoning and code generation while shrinking the model from ~53 GB (fp16) to 11 GB. Prose quality is the only meaningful regression; use a different model for writing tasks.

Config: server on port 8082, full 262K context, MTP speculation, q8_0 KV. Default for all benchmarking going forward.