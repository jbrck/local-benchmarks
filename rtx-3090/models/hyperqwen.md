# HyperQwen [Qwen3.8-27B + DFlash2]

**TL;DR:** Rejected. The HyperQwen results were three mislabeled prose files from other models; no genuine battery data exists under this name.

**Role:** Math specialist — highest local 27B MATH-500 score by 4.4 points. Fastest local 27B by 3x.

## Results

| Test | Score | Local 27B rank |
|---|---|---|
| MATH-500 | **94.6%** (473/500) | 🏆 #1 |
| HumanEval+ | 83.5% (137/164) | #5 |
| GPQA-Diamond | 49.0% (97/198) | #3 |
| Refusal HARMFUL | 15/20 (75%) | — |
| Refusal BENIGN (over-refusal) | 2/30 (7%) | — |
| Reasoning bench | **100%** (8/8) | 🏆 #1 |

## Timing & Speed

| Test | Total | Per item | Speed |
|---|---|---|---|
| MATH-500 (500 items) | **66 min** | 7.9s | 162 tok/s avg |
| HumanEval+ (164 items) | **18 min** | 6.7s | — |
| GPQA-Diamond (198 items) | **~90 min** | 27s (incl. thinking) | — |

## Speed

| Metric | Value |
|---|---|
| Throughput | **162 tok/s avg** (MATH-500 benchmark run) |
| Peak tok/s | 195-220 tok/s (simple prompts) |
| Context | 41k tokens (DFlash2 limits KV cache to 5 GiB) |
| VRAM | 23.7 GB (pinned, zero headroom) |

## Architecture

- **Engine:** vLLM 0.9.0+ with DFlash2 speculative decoding (15-token draft)
- **Main model:** Qwen3.8-27B (AWQ 4-bit, W4A16)
- **Draft model:** Qwen3.8-27B-DFlash2-W4A16 (lightweight prediction head)
- **KV cache:** bfloat16, auto-sized via GPU_UTIL=0.92
- **Reasoning:** Qwen3 parser enabled (`reasoning` field, not `reasoning_content`)
- **Prefix caching:** Enabled
- **Attention:** FLASH_ATTN backend

## Comparison vs local 27B field

| Model | Engine | MATH-500 | HumanEval+ | GPQA-D | tok/s |
|---|---|---|---|---|---|
| **HyperQwen** | vLLM DFlash2 | **94.6%** | 83.5% | 49.0% | **162** |
| swift-qwen3.8-27b | llama.cpp | 89.2% | 87.8% | 50.5% | ~55 |
| qwen3.8-27b GGUF | llama.cpp | 86.8% | **91.5%** | 46.0% | ~55 |
| OrcaSAQ-2-27B | vLLM | 90.8% | 89.6% | 55.0%* | 11.6 |
| Holo4-27B | llama.cpp | 73.4% | 90.2% | **56.1%** | 23.1 |

## Notes

HyperQwen is DFlash2 — two models on one card: a 27B main model and a smaller draft model that predicts the 27B's next 15 tokens. When the draft is right, the 27B validates all 15 in one parallel pass. When wrong, it regenerates. This gives **162 tok/s average** — 2-3x faster than llama.cpp GGUF 27Bs, and nearly matches Spark (a 4B model) at 185 tok/s.

**MATH is the standout.** 94.6% is the highest local 27B score by 4.4 points. The 8192-token budget was mandatory — at 4096 the model's thinking mode consumed all tokens before producing content, scoring 85% pilot / 37% GPQA. Bumping to 8192 fixed it.

**Coding is the weakness.** 83.5% HumanEval+ is bottom of the 27B pack. Thinking mode hurts code — the model reasons before outputting, and the bench reads `content` not `reasoning` for extraction.

**GPQA is middle of the pack.** 49.0% at full 198 problems. Competitive with swift (50.5%) and above qwen GGUF (46.0%). The 27s average includes thinking time on hard problems that burn all 8192 tokens.

**Refusal behavior is balanced.** 75% harmful refusal (15/20) with only 7% over-refusal on benign (2/30). Below the over-cautious models (swift: 100% harmful refusal at 17% over-refusal) but well above uncensored models (OrcaSAQ: 20% harmful refusal).

## Key Findings

- **DFlash2 trades context for speed.** The draft model burns ~4 GiB that would be KV cache. Context drops from 57k (vanilla vLLM) to 41k. Worth it for benchmarks, limiting for long-context RAG.
- **No GPU headroom.** 23.7 of 24 GiB used. Nothing else can run on the GPU while HyperQwen is loaded.
- **max_tokens must be 8192 for thinking models.** Bench scripts default to 4096. Thinking models burn the full budget on reasoning and never reach `content`. This cost ~9% on GPQA and ~10% on MATH.
- **LiteLLM auth key is `sk-litellm-...`** — stored in `~/.hermes/config.yaml` and `/mnt/data/litellm/.env`.
- **Full battery runtime: ~3 hours** (MATH 66m + GPQA 90m + HE+ 18m + prose/refusal/reasoning ~5m).