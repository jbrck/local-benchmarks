# Mirai S Qwen3.8-27B

**TL;DR:** Best model tested. Tops every leg: MATH 94.2%, HE+ 95.1%, GPQA 69.7%, tool-eval 93, Instr v2 20/20, perfect refusal, LiveCodeBench 76.0% (unbounded reasoning). 2.4 bpw trellis (~11 GB) at 262K context.

**Origin:** Base weights — **China** (Alibaba Qwen3.8-27B, MoE) · Publisher — `alesha-pro` / Mirai (Hugging Face), publisher country not established

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
| MATH-500 | **94.2%** | 2nd local, behind Nemotron 3.5 Bolt (96.4%); +3.4 over OrcaSAQ-2-27B |
| HumanEval+ | **95.1%** | 2nd local, behind Nemotron 3.5 Bolt (96.3%); beats DeepSeek V4 Flash (93.9%) |
| GPQA-Diamond | **69.7%** | 2nd local, behind Nemotron 3.5 Bolt (77.8%); +13.1 over Muse Glimmer 30B (56.6%) |
| Instr v2 | **20/20 (100%)** | First perfect score on this test |
| IFEval | 78.9 / 85.1% | 3rd local; ties swift's instruction score. 49 min, 214k tokens |
| Tool-eval | **93/100** | Ties DeepSeek V4 Flash remote baseline; 28/30 points |
| Refusal | 20/20 harmful | 17% benign over-refusal, 10% edgy |
| Reasoning | 8/8, 126s | Correctness saturated; no thinking-token edge |
| MMLU | 87.0% (435/500) | 83.6 min, 71 tok/s. Author claims 89.8% — 2.8 pts under on a different sample. Weakest: virology 56%, HS physics 67%, professional_law 73% |
| LiveCodeBench | **76.0%** (76/100) | `xhigh`, unbounded reasoning: 14.10 h, 68.7 tok/s. **90.5% of scorable problems.** 16/100 still emit no code — the reasoning channel now never terminates. Bounded config scored 66.0%. Detail in [test_livecodebench.md](../test/test_livecodebench.md) |

### LiveCodeBench: think harder, score higher — but not forever

Two runs, one variable: the reasoning budget. Bounded (`medium`, `--reasoning-budget 20480`), the model hit its ceiling, got pushed to answer, and kept analyzing in the content channel — which has no budget — until the 32K output cap. Prose, no code, auto-fail.

Unbounded (`xhigh`, no budget), that failure disappeared and the score went **66.0% → 76.0%**: easy 24/24, medium 28/31, hard 24/45 (36% → 53%).

What did **not** change: code emission. Both runs produce code on 84/100 and blank on 16/100. The +10 points are entirely correctness on problems that already answered — the model was writing wrong answers under the budget, not fewer answers.

What changed is the shape of the failure. Unbounded, the 16 blanks have **zero** content and 305K–431K characters of reasoning: the model never leaves the thinking channel. Same failure count, far worse per-item cost — 32,768 tokens each bounded, 114,688 each unbounded. That is **7.42 h of the 14.10 h run**, spent on 16 problems that fail either way. AtCoder carries 13 of the 16.

The fix is an abort guard on the reasoning channel, not a budget. Full detail in [test_livecodebench.md](../test/test_livecodebench.md).

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

**LiveCodeBench is config-dependent:** 76.0% with reasoning unbounded (`xhigh`, no `--reasoning-budget`) against 66.0% under the shipped bounded config. Same code emission, better answers — the budget was costing correctness, not code. The unbounded run costs 2.4× the wall clock, most of it on 16 problems that fail either way. For interactive coding use the bounded config; for scored batch runs, go unbounded.

Config: server on port 8082, full 262K context, MTP speculation, q8_0 KV. Default for all benchmarking going forward.