# Mirai S Qwen3.8-27B

**TL;DR:** Best model tested. Tops every leg: MATH 94.2%, HE+ 95.1%, GPQA 69.7%, tool-eval 93, Instr v2 20/20, perfect refusal. 2.4 bpw trellis (~11 GB) at 262K context.

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
| MMLU | 87.0% (435/500) | 83.6 min, 71 tok/s. Author claims 89.8% — 2.8 pts under on a different sample. Weakest: virology 56%, HS physics 67%, professional_law 73% |
| LiveCodeBench | 66.0% (66/100) | 5.89 h, 70 tok/s. **78.6% of scorable problems** — 16/100 never emitted code (hard problems, reasoning overran the 32K content budget) |

### LiveCodeBench: the non-termination penalty

Hard problems expose a real failure mode. The model thinks in the reasoning channel (~20K tokens, the server's `--reasoning-budget` ceiling), then writes a short code block. On ~25% of hard problems the budget fires, the server pushes it to answer, and it keeps analyzing — in the content channel, which has no budget. It burns the output cap on prose and never writes code, so the item auto-fails.

Breakdown: easy 24/24, medium 26/31, hard 16/45. AtCoder is where it hurts (41/68 = 60%, 14 of the 16 no-code failures). Of the 84 problems that did produce code, 66 passed = 78.6%. Raise the output budget and it buys more analysis, not more solutions. Full detail in [test_livecodebench.md](../test/test_livecodebench.md).

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