# GLM-5.3-Flash-Maya-S24

**TL;DR:** Runs on Badger-001, but is **not benchmarkable** — 4.5 tok/s decode and 8.7 tok/s prefill. No battery was run. Recorded as a speed-limited probe so the hardware ceiling is on file, not guessed.

**Origin:** Base weights — **China** (zai-org GLM-5.3-Flash) · Publisher — community (`peasantsmith`, Project Maya)

**Role:** Capacity-tier experiment. Proves a 321B MoE can load and run on one 3090 plus 62 GB of RAM. Too slow for evaluation, too slow for chat.

## Results

**None.** No leaderboard row. Every battery leg is infeasible at this speed (see Cost projection). Any number produced here would measure the disk, not the model.

## Timing & Speed

| Metric | Value |
|---|---|
| Decode (steady) | **4.50 tok/s** |
| Prefill | **8.7 tok/s** |
| Time to first token, 294-tok prompt | **34 s** |
| Decode on a 294-tok LCB prompt | 3.24 tok/s |
| Load time | ~110 s |
| VRAM | 22.3 / 24.5 GB |
| Swap touched | ~5 GB |
| File size | 94.7 GB (88.2 GiB), 3 shards |

Both numbers move with expert placement:

| Expert placement | Decode | Prefill | VRAM |
|---|---|---|---|
| All 46 layers on CPU (`--n-cpu-moe 46`) | 3.37 tok/s | 5.1 tok/s | 8.2 GB |
| 8 layers' experts on GPU (`--n-cpu-moe 38`) | **4.50 tok/s** | **8.7 tok/s** | 22.3 GB |

`--n-cpu-moe 38` is the ceiling. One more layer OOMs a 24 GB card.

## Architecture

- **Base:** GLM-5.3-Flash — 321B total, ~18B active MoE, MIT
- **Quant:** IQ2_XXS_S experts, Q4_K attention and shared experts, F32 router
- **Layout:** 46 layers, 288 experts / 8 active, hidden 4096, expert FFN 2048, MLA KV 256-dim, 1M native context, 3 leading dense blocks
- **GGUF arch:** `glm5-next` — registered in llama.cpp build 11670, loads clean
- **MTP:** the NextN layer is present in the file but this build logs it as an unused tensor, so no speculative decoding
- **Engine:** llama.cpp, `-ngl 99 --n-cpu-moe 38`, q8_0 KV, 8192 ctx

The Maya-S24 point is deliberate: Project Maya's S tier keeps attention and shared experts at Q6_K; S24 drops them to Q4_K to buy ~1.5 GB more expert room on the card. That is a decision made for 24 GB GPUs specifically, and it works — it is why 8 expert layers fit on the 3090 instead of 6.

## Cost projection (why no battery)

| Test | Per problem | Full leg |
|---|---|---|
| LiveCodeBench (100) | ~8.3 min | **~13.8 h** |
| MATH-500 (500) | ~30 min | **~253 h** |
| MMLU (~14k) | ~28 s | **~109 h** |

Liberator: 294-tok prompt at 8.7 tok/s prefill is 34 s, plus ~1,500 generated tokens at 3.24 tok/s. Even LCB alone lands above the 8 h guard, so it would truncate before finishing.

## Key Findings

- **The 88.2 GiB file does not fit.** 62 GB RAM + 24 GB VRAM = 86 GB, and the OS needs several of that. About 5 GB of swap is in active use while running. It works, but it is page-cache roulette, not residency.
- **Expert locality is doing real work.** Decode climbed 1.96 → 3.37 → 4.50 tok/s across successive requests as the hot expert set migrated into VRAM and page cache. Cold, this model is slower than the steady-state number suggests.
- **GPU-resident experts pay off 3.4x.** Moving 8 of 43 MoE layers from CPU to GPU took prefill from 5.1 to 8.7 tok/s. The 8700K has no AVX-512, so every expert computed on CPU is on the slow path; the GPU has headroom the CPU does not.
- **Prefill is the binding constraint, not decode.** 8.7 tok/s means a 3,000-token agent prompt costs 6 minutes before the first token. Any benchmark with long prompts is dominated by this.
- **Quality is intact at 2 bits.** Correct arithmetic with a clean reasoning trace on the first try. The quant is sound; the hardware is the problem.
- **Not for daily use.** `run.sh` in the model directory relaunches it. It must not run alongside ComfyUI — 22.3 GB of VRAM leaves nothing.
