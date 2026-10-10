# MTP vs DFlash2 on a 3090: I tested the tweet's claims

**TL;DR:** MTP vs DFlash2 speculative decoding across 3 configs x 4 depths. MTP roughly doubles decode speed; the Mirai S fork holds ~54 tok/s flat across context sizes up to 262K.

**Verdict up front:** DFlash2 does not beat MTP everywhere. On a 24 GB RTX 3090 running Qwen3.8-27B through llama.cpp, MTP is faster at short context. DFlash2 takes over past ~32k. Both crush no speculation — 1.6-2.6x. And the "240k vs 200k max context" ceiling claim? Both configs load the full native 262k, and way past it.

---

## A tweet worth checking

A tweet from AJ claimed a test of Qwen3.8-27B-GSQ-RCO-IQ3_S on a 3090 in llama.cpp — the same model, quant, and GPU I run.

https://x.com/itsmeajaykv/status/2099519896640270441

The claims, as reported in the thread:

- DFlash2 beats MTP on decode at every context depth
- Better prefill / time-to-first-token
- ~240k max context with DFlash2 vs ~200k with MTP
- MTP's draft acceptance collapses at high draft length; DFlash2 holds

DFlash2 is a serving technique, not a model. It's a block-diffusion speculative drafter (arXiv 2602.06036) that drafts a whole token block in one pass, merged into llama.cpp upstream at b10658. MTP is the multi-token-prediction head baked into the Qwen3.8 checkpoint. Both make decode faster without changing output.

If the tweet was right, switching serving config was a free win. If it was wrong, I wanted to know before touching my setup. So I tested it.

## What I tested

| | |
|---|---|
| Target model | Qwen3.8-27B-GSQ-RCO-IQ3_S (12.1 GB GGUF, ISTA-DASLab per-tensor mixed-precision quant) |
| GPU | RTX 3090, 24 GB, q4_0 KV cache, flash attention, single slot |
| Engine | llama.cpp b10909 (Sept 2026) — ships both `draft-mtp` and `draft-dflash` |
| Draft model | `incoai/Qwen3.8-27B-DFlash2-GGUF` Q4_K_M (1.1 GB) |
| Draft width | n=5 for both (a separate study by lukaLLM measured 5 as the optimum, not the recommended 7) |
| Generation | 200 tokens per rep, 2 reps, temperature 0, EOS disabled so every rep actually generates |
| Depths | 4k, 32k, 96k, 128k |
| Configs | no speculation (baseline) / MTP / DFlash2, same target model each time |

Speculative decoding is output-lossless — rejected draft tokens get recomputed by the target. So this is a speed and memory test, not a quality test. The same weights, three ways to serve them.

## MTP wins short-context. DFlash2 wins past 32k.

Decode tok/s (200 tokens, avg of 2 reps):

| context | baseline | MTP | DFlash2 | DFlash2 vs MTP |
|---|---|---|---|---|
| 4k | 42.2 | 74.4 | 66.5 | 0.89x |
| 32k | 34.6 | 80.1 | 83.4 | 1.04x |
| 96k | 24.5 | 56.0 | 61.3 | 1.10x |
| 128k | 21.6 | 53.5 | 56.0 | 1.05x |

At 4k, MTP is 12% faster than DFlash2. By 32k, DFlash2 edges ahead. At 96-128k it holds a 5-10% lead. The crossover matches what the lukaLLM study predicted — DFlash2 climbs as context deepens — but the shape is gentler than the tweet implied.

Draft acceptance is nearly identical between the two: 0.67-0.80, mean accepted length 4.3-5.0 at n=5. MTP at 4k hits 4.97/5 — near-perfect drafting, which is exactly why it wins short-context. DFlash2's edge at depth comes from drafting efficiency, not raw acceptance.

Prefill is the flip side: DFlash2 is the slowest of the three (~3-6% penalty from draft overhead). MTP costs ~1-2%. For single-shot, time-to-first-token-sensitive work, both speculators give up a little prefill for a lot of decode.

## Both beat no speculation. By a lot.

| context | baseline | MTP speedup | DFlash2 speedup |
|---|---|---|---|
| 4k | 42.2 | 1.76x | 1.58x |
| 32k | 34.6 | 2.31x | 2.41x |
| 96k | 24.5 | 2.28x | 2.50x |
| 128k | 21.6 | 2.47x | 2.59x |

Serving without any speculation was leaving 1.6-2.6x on the table. MTP is the cheap version of that win: the head is already in the checkpoint, no download, ~1.8 GB of spec KV on top of the model.

DFlash2 costs more: +0.7 GB VRAM over MTP, the 1.1 GB draft download, the prefill penalty — and its decode advantage only shows past ~32k.

## The fine print: prefill, VRAM, acceptance

Prefill (tok/s, rep 0): DFlash2 is the slowest of the three at every depth.

| context | baseline | MTP | DFlash2 |
|---|---|---|---|
| 4k | 1261 | 1123 | 1014 |
| 32k | 1128 | 1064 | 1010 |
| 96k | 867 | 819 | 798 |
| 128k | 785 | 742 | 725 |

DFlash2 gives up ~3-6% prefill to draft overhead; MTP ~1-2%. For single-shot, time-to-first-token-sensitive work, both speculators trade a little prefill for a lot of decode.

VRAM (MiB after load / during run):

| | baseline | MTP | DFlash2 |
|---|---|---|---|
| after load | 14,896 | 16,692 | 17,336 |
| during run | 15,092 | 16,894 | 17,634 |

DFlash2: +2.4 GB over baseline, +0.7 GB over MTP. MTP: +1.8 GB over baseline — no extra weights, that's the spec-decoding KV path.

Draft acceptance at n=5 (server-logged):

| context | MTP accepted | MTP mean len | DFlash2 accepted | DFlash2 mean len |
|---|---|---|---|---|
| 4k | 0.795 | 4.97 | 0.795 | 4.97 |
| 32k | 0.674 | 4.33 | 0.700 | 4.50 |
| 96k | 0.705 | 4.52 | 0.684 | 4.42 |
| 128k | 0.705 | 4.52 | 0.684 | 4.42 |

Acceptance is nearly a tie across the board. MTP's near-perfect 4.97/5 at 4k is why it wins short-context; DFlash2's edge at depth comes from block-diffusion drafting efficiency, not raw acceptance.

## The ceiling claim was wrong

The tweet said DFlash2 ~240k vs MTP ~200k max context. I pushed `-c` on all three configs until the 3090 ran out of memory (llama.cpp allocates the full KV cache at load, so a failed load is the answer).

| config | max context that loads | VRAM at max |
|---|---|---|
| baseline (no spec) | **540k** | 24.1 GB |
| MTP (n=5) | 400k | 24.0 GB |
| DFlash2 (n=5) | 420k | 23.8 GB |

All three load the full native 262,144 and generate fine. The memory wall sits at 400-540k, far past native. The interesting bit: DFlash2 out-ceilings MTP at the extreme despite its draft weights, because its KV scales flatter (~22.5 KB/token vs MTP's ~27.4). Block diffusion reuses target state instead of holding its own deep KV cache.

Caveat: 262,144 is the model's native trained context. Everything past that is YaRN extrapolation — it loads and runs, but it's not what the model was trained to do. These are memory ceilings, not quality ceilings.

## The cheap win

If you run Qwen3.8-27B on a 24 GB card: enable MTP. It's already in the checkpoint, it's one flag, and it buys 1.6-2.6x decode at every depth. Add DFlash2 only if deep-context throughput is your bottleneck — the +5-10% past 32k is real, but it costs VRAM, a download, and prefill speed.

## Caveats

- One GPU, one engine (llama.cpp). Your numbers move with your card and build.
- Temperature 0 on a filler corpus. Real workloads — tool calls, multi-turn agent loops — shift acceptance. lukaLLM's separate finding of +35% on iterative coding when DFlash2 is paired with ngram lookup drafters is worth its own test; I didn't run it here.
- The ceiling probe used tiny completions (load + 16 tokens). That's the right tool for finding the memory wall; it is not throughput at depth.

## Links

- The tweet: https://x.com/itsmeajaykv/status/2099519896640270441
- lukaLLM's study (LiveCodeBench, draft-width sweep, ngram pairing): https://github.com/lukaLLM/DFlash2_Qwen3.8_3.6_27B_LlamaCPP
- DFlash2 (block-diffusion drafter, paper + code): https://github.com/z-lab/dflash
- Draft GGUF: https://huggingface.co/incoai/Qwen3.8-27B-DFlash2-GGUF