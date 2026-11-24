# Context-depth speed

**TL;DR:** How fast a model decodes and prefills as the prompt grows, and the deepest prompt the server will take. Run by the `ctxspeed` leg of [`jbrck/local-model-battery`](https://github.com/jbrck/local-model-battery). Inkling 2 Mini on Project Maya decodes at **123.6 tok/s at 512 tokens and 74.6 tok/s at 237K**, with no cliff in between.

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

## Copy task

| Depth | Valid trials | Decode tok/s | Copy ratio |
|---|---|---|---|
| 512 | 1 of 2 | 189.0 | 0.5 |
| 8,192 | 1 of 2 | 246.7 | 1.0 |
| 32,768 | 2 of 2 | 226.7 | 1.0 |
| 65,536 | 2 of 2 | 166.2 | 0.2 |
| 131,072 | 1 of 2 | 135.5 | 0.3 |
| 237,326 | 1 of 2 | 114.2 | 0.1 |

The drafter reaches 247 tok/s when the model copies cleanly, and the copy ratio swings between 0.0 and 1.0 with no pattern in depth. Use the prose column for any decode comparison.

## Caveats

- One model, one window. The ceiling here is the 240K launch setting.
- Two trials per cell. The prose spread between the two trials is under 6%, but two is a small sample.
- The 10% depth tolerance marks some shallow trials invalid. The leg resizes a prompt once when it lands off its depth. A second resize would recover most of them.
- Maya single stream only.

Source: `local-model-battery` at `de5596c`.
