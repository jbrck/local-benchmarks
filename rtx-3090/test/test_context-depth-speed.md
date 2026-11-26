# Context-depth speed

**TL;DR:** How fast a model decodes and prefills as the prompt grows, and the deepest prompt the server will take. Run by the `ctxspeed` leg of [`jbrck/local-model-battery`](https://github.com/jbrck/local-model-battery). Inkling 2 Mini on Project Maya decodes at **123.6 tok/s at 512 tokens and 74.6 tok/s at 237K** with a 240K window, and takes a **1,015,652-token prompt** with a 1M window, where decode falls to 6.5 tok/s and the first token takes 41 minutes.

## What the leg does

The leg climbs a ladder of eight prompt depths (512 up to 1,048,576 tokens) and stops at the first rung the server refuses, clips, times out on, or crashes at. It then bisects between the last good rung and the failed one to within 4,096 tokens. The result carries a `ceiling` block: deepest depth that worked, first depth that failed, and why.

Before the climb it reads the served window from the server's `/props` and holds back any rung that does not fit. That saves the hours a certain-to-fail prefill would burn. `--ignore-window` skips the check. The window is a launch setting, not the hardware limit. To find the hardware limit, relaunch the server with a larger window and rerun.

Two tasks run at each depth, two trials each:

- **prose:** the prompt is filler text the model is told to ignore, followed by a request for a long essay on the printing press. Output length does not depend on the filler, so this measures real decode speed at that depth.
- **copy:** the model is asked to reproduce the final 400 words of the filler word for word. The answer is already in the prompt, so this shows how much a speculative drafter gains on copied text. The numbers are noisy and are not a decode rate.

Filler is real text from the corpus. Past one corpus pass, later passes reshuffle the words inside each piece so a drafter cannot win on repetition. Prefill is timed apart from decode (time to first token). Seed 42, temperature 0, thinking off, 1,024-token decode window.

## Results: Inkling 2 Mini (UD-Q4_K_XL, Project Maya)

Served at 245,760 context (240K), drafter on. Hardware: [RTX 3090 24 GB](../README.md#system), power-limited to 300 W.

| Depth (tokens) | Prompt tokens | Decode tok/s | Decode vs 512 | Prefill tok/s | Time to first token |
|---|---|---|---|---|---|
| 512 | 531 | 123.6 | 100% | 1,048 | 0.5 s |
| 8,192 | 8,195 | 119.5 | 97% | 2,091 | 3.9 s |
| 32,768 | 33,139 | 110.3 | 89% | 2,026 | 16.4 s |
| 65,536 | 66,410 | 101.8 | 82% | 1,881 | 35.3 s |
| 131,072 | 133,401 | 89.4 | 72% | 1,637 | 81.5 s |
| 237,326 | 237,154 | **74.6** | **60%** | 1,362 | 174.2 s |

Prose task, median of valid trials. Decode loses 40% over the window. Prefill peaks near 8K and falls 35% by 237K.

**Ceiling:** `served_window`. Every rung the 240K window allows ran, and nothing failed. The 262,144, 524,288 and 1,048,576 rungs were held back as past the window. The hardware limit is not measured.

**Run:** 24 trials, 22 minutes, 19 valid. Five trials were marked invalid: four copy trials where the model stopped after 15 to 94 tokens, and one prose prompt that landed 14.6% over its depth at the 512 rung. No timeouts, no cap hits.

## Copy task, 240K window

| Depth | Valid trials | Decode tok/s | Copy ratio |
|---|---|---|---|
| 512 | 1 of 2 | 189.0 | 0.5 |
| 8,192 | 1 of 2 | 246.7 | 1.0 |
| 32,768 | 2 of 2 | 226.7 | 1.0 |
| 65,536 | 2 of 2 | 166.2 | 0.2 |
| 131,072 | 1 of 2 | 135.5 | 0.3 |
| 237,326 | 1 of 2 | 114.2 | 0.1 |

The drafter reaches 247 tok/s when the model copies cleanly, and the copy ratio swings between 0.0 and 1.0 with no pattern in depth. Use the prose column for any decode comparison.

## Results: 1M window (the ceiling run)

The 240K run stopped at its own window, so Maya was relaunched with `--max-context 1048576` and the leg rerun. Same model, same drafter, same 3090. Nothing failed: every rung the window allows ran, from 512 to 1,016,057 tokens. The 1,048,576 rung was held back because the wrapper would push it past the window.

| Depth (tokens) | Prompt tokens | Decode tok/s | Decode vs 512 | Prefill tok/s | Time to first token |
|---|---|---|---|---|---|
| 512 | 531 | 89.5 | 100% | 226 | 2.3 s |
| 8,192 | 8,195 | 87.6 | 98% | 705 | 11.7 s |
| 32,768 | 33,139 | 83.2 | 93% | 743 | 44.6 s |
| 65,536 | 66,410 | 77.9 | 87% | 732 | 90.7 s |
| 131,072 | 133,401 | 57.8 | 65% | 692 | 192.9 s |
| 262,144 | 262,192 | 28.2 | 32% | 626 | 418.8 s |
| 524,288 | 523,547 | 15.6 | 17% | 535 | 979.2 s |
| 1,016,057 | 1,015,652 | **6.5** | **7%** | 412 | 2,462 s (41 min) |

Prose task, median of valid trials. The 512 prefill figure is not a rate: the prompt is too short to time.

**Ceiling:** `served_window`, 1,015,652 prompt tokens. The hardware limit is still not measured, because the window is the cap Maya was launched with. The model took everything the window allowed.

**Run:** 32 trials, 4 hours 51 minutes, 26 valid. Six trials were marked invalid: five copy trials where the model stopped after 15 to 94 tokens, and one prose prompt that landed 14.6% over its depth at the 512 rung. No timeouts, no cap hits.

### What the big window costs

The same model decodes slower everywhere on the 1M window than on the 240K one.

| Depth | 240K window | 1M window | Loss |
|---|---|---|---|
| 512 | 123.6 tok/s | 89.5 tok/s | 28% |
| 8,192 | 119.5 | 87.6 | 27% |
| 32,768 | 110.3 | 83.2 | 25% |
| 65,536 | 101.8 | 77.9 | 23% |
| 131,072 | 89.4 | 57.8 | 35% |

Prefill drops from about 2,000 to about 700 tok/s over the same range. Decode also steps down harder past 66K on the 1M window (78 to 58 tok/s between 66K and 133K), where the 240K window glides.

GPU memory in use was 23.2 GB on the 1M window, about the same as the 240K window, so a 1M-token cache cannot sit wholly in VRAM. My read is that part of it spills to system RAM, which would explain both the flat loss and the steep fall past 131K. I did not confirm that. At the end of the run the host showed 5 of 5 GB swap used and 1 GB of free RAM. That is an observation, not a cause.

Use the 240K numbers for how the model serves day to day. Use the 1M numbers to see how far the 3090 stretches and what it costs.

### Copy task, 1M window

| Depth | Valid trials | Decode tok/s | Copy ratio |
|---|---|---|---|
| 512 | 1 of 2 | 88.9 | 1.0 |
| 8,192 | 1 of 2 | 87.2 | 1.0 |
| 32,768 | 2 of 2 | 83.2 | 1.0 |
| 65,536 | 2 of 2 | 77.8 | 0.1 |
| 131,072 | 1 of 2 | 63.6 | 0.1 |
| 262,144 | 1 of 2 | 21.7 | 0.9 |
| 524,288 | 1 of 2 | 15.7 | 0.8 |
| 1,016,057 | 2 of 2 | 6.2 | 0.9 |

Copy decode matches prose at every depth. The 240K window's drafter gain on copied text (up to 247 tok/s) is gone here.

## Caveats

- One model. Both ceilings are launch settings (240K and 1M). The hardware limit is past 1M and unmeasured.
- Two trials per cell. The prose spread between the two trials is under 6%, but two is a small sample.
- The 10% depth tolerance marks some shallow trials invalid. The leg resizes a prompt once when it lands off its depth. A second resize would recover most of them.
- Maya single stream only.

Source: `local-model-battery` at `de5596c`. 240K run tag `live-240k`, 1M run tag `live-1m`.
