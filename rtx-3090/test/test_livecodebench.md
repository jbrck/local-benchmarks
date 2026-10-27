# LiveCodeBench

**TL;DR:** 100 competition coding problems with hidden test cases, drawn from a contamination-free window. Mirai S Qwen3.8-27B scores **76.0%** when the server lets it think without a budget — up from **66.0%** under the model's default bounded config. Code emission is identical at 84/100 in both runs, so the +10 points are entirely correctness: the same problems, answered right more often. The cost is wall clock — 14.1 h against 5.9 h.

**LFM2.5-24B-A2B** ran the same 100 problems and produced code on **99 of 100** — the highest emission recorded here — while scoring **61.0%** in **3.07 h**. It writes code where Mirai doesn't, and gets it wrong where Mirai gets it right.

**Nemotron 3.5 Bolt 74B-A9B has no number here.** Its run was killed at 54 of 100 problems generated and never scored: 19 of the 54 produced no code, and 84% of generated tokens went to items that returned nothing. At the observed rate it projected past 20 hours, against 14.1 h for Mirai S. That gap is the point — this is Mirai S's headline win, and it is the one leg Bolt has no data for.

**Nanbeige4.3-33B has no number here either.** Its run was killed at 55 of 100 problems generated and never scored: 11 of the 55 produced no code, with thinking-channel output up to 106k chars on items that emitted zero code at temp 0. At ~9 min/problem it exhausted the 8 h budget before generation finished.

**What it evaluates:** Competition programming problems from LeetCode, AtCoder, and Codeforces, each with hidden test cases run against the generated solution. Problems are date-stamped, so results can be filtered to a contamination-free window. This is a harder coding test than HumanEval+ — algorithmic thinking on unseen problems, not boilerplate completion.

**Reading scores:** 60-70% is solid for a 27B on hard problems. The easy and medium split is where a capable model lives; the hard tail separates models. Watch the no-code rate — a model that reasons well but doesn't terminate scores low for the wrong reason, and that is this model's entire story.

## Results

Two runs, one variable: the reasoning budget.

| Config | Accuracy | Scorable pass | Code emitted | Wall clock |
|---|---|---|---|---|
| Bounded — `medium`, budget 20,480 | 66.0% (66/100) | 66/84 = 78.6% | 84/100 | 5.89 h |
| **Unbounded — `xhigh`, no budget** | **76.0% (76/100)** | **76/84 = 90.5%** | 84/100 | 14.10 h |

Both runs leave exactly 16 problems with no code. Same failure count, completely different failure mechanism.

## Removing the budget changed the failure, not the success rate

The bounded run had a two-channel problem. `--reasoning-budget 20480` capped the reasoning channel; when the model hit it, the server injected an answer prompt and pushed the model into the content channel — which had no budget. It kept analyzing there, in prose, until the 32K output cap. No code.

The unbounded run removed the ceiling. That eliminated the content-channel failure entirely — and replaced it with pure non-termination.

| | Bounded | Unbounded |
|---|---|---|
| Content on solved items | 62–48,896 chars | 62–5,162 chars |
| Reasoning on solved items | 134–80,619 chars | 141–359,692 chars |
| Content on no-code items | **12,865–43,638 chars** | **0** |
| Reasoning on no-code items | 22,618–75,962 chars | **305,723–431,560 chars** |
| Tokens on no-code items | 32,768 (all 16) | 114,688 (all 16) |

Bounded failures end mid-sentence in prose — the content tail of one reads `"...DP"`. Unbounded failures have **zero** content and 300K+ characters of reasoning: the model never leaves the thinking channel, burns the full 114,688-token cap, and returns nothing.

The success path also changed. Unbounded successes emit remarkably tight code — the 90th-percentile content block is under 5,200 characters, because the model has finished thinking before it starts writing. Bounded successes sometimes ran to 48,896 characters, because the model was pushed into answering while still mid-analysis.

## By difficulty

| Difficulty | Bounded | Unbounded |
|---|---|---|
| easy | 24/24 = 100% | 24/24 = 100% |
| medium | 26/31 = 84% | 28/31 = 90% |
| hard | 16/45 = 36% | 24/45 = 53% |

Easy is solved. Medium gained 2. Hard gained 8 — a 17-point jump on the tier that was carrying all the failures.

## By platform

| Platform | Bounded | Unbounded | No-code (bounded → unbounded) |
|---|---|---|---|
| leetcode | 25/32 = 78% | 27/32 = 84% | 2 → 3 |
| atcoder | 41/68 = 60% | 49/68 = 72% | 14 → 13 |

Non-termination is an AtCoder problem: 13 of the 16 no-code items in the unbounded run. AtCoder's harder, more mathematically-stated problems are what push the model into unbounded thinking.

## Where the time goes

| | Bounded | Unbounded |
|---|---|---|
| Wall clock | 5.89 h (21,196 s) | 14.10 h (50,777 s) |
| Generation tokens | 1,490,818 | 3,486,964 |
| Throughput (wall) | 70.3 tok/s | 68.7 tok/s |
| Median tokens/problem | 13,844 | 16,693 |
| Items hitting the cap | 25 | 16 |
| Tokens in capped items | 819,200 | 1,835,008 |
| **Time in capped items** | **3.24 h = 55%** | **7.42 h = 53%** |

Throughput is unchanged. The run is 2.4× longer only because it generated 2.34× the tokens. Over half of *both* runs is spent on problems the model never solves.

The number that matters: in the unbounded run the 16 runaway items alone took **7.42 h — longer than the entire bounded run (5.89 h)**. Strip the runaways from each and the real comparison is:

| | Real work | Score |
|---|---|---|
| Bounded | 5.89 − 3.24 = **2.65 h** | 66.0% |
| Unbounded | 14.10 − 7.42 = **6.68 h** | 76.0% |

The 10 points cost about 4 extra hours of *legitimate* thinking time (the new config makes solvable hard problems think longer before answering), plus 4.2 extra hours of pure waste.

## Verdict

Raising the ceiling bought correctness on problems that already produced code — it did not buy more code. Net +10 points overall, +17 on hard, at 2.4× wall clock.

The economics are poor only because of the runaways. An abort guard that kills a problem once the reasoning channel passes some threshold — say 100K characters with no code in sight — would have recovered most of the 7.42 h while keeping nearly all of the gain. That is the config this model actually wants: **xhigh with a runaway detector**, not xhigh alone and not a hard budget. A 76% run in roughly 7 hours instead of 14.

For interactive coding use, the bounded config is the better default — it is 2.4× faster and loses only 10 points. For batch evaluation where hours are cheap and scores matter, unbounded wins.

## LFM2.5-24B-A2B — the most code emitted, the least of it correct

LFM2.5-24B-A2B ran the identical 100 problems under a single unbounded config. It produced a code block on **99 of 100** — the highest emission on record here, 15 better than Mirai — and still finished 15 points behind.

| | LFM2.5-24B-A2B | Mirai S (xhigh) |
|---|---|---|
| Score | 61.0% (61/100) | 76.0% (76/100) |
| Scorable | 61/99 = 61.6% | 76/84 = 90.5% |
| Code emitted | **99/100** | 84/100 |
| Wall clock | **3.07 h** | 14.10 h |
| Generation tokens | 1,918,672 | 3,486,964 |
| Throughput | **191 tok/s** | 68.7 tok/s |
| easy | 24/24 = 100% | 24/24 = 100% |
| medium | 23/31 = 74% | 28/31 = 90% |
| hard | 14/45 = 31% | 24/45 = 53% |

The failure modes are opposites. Mirai stops thinking and never writes code — 16 blanks, each carrying 305K+ characters of reasoning. LFM2.5 always writes code, and the code is wrong on two-thirds of the hard tier.

For a batch harness LFM2.5's failure is the cheaper one. It terminates, it returns something gradeable, and the entire run costs 3.07 h against Mirai's 14.10.

Runaway behaviour is present but mild: one item hit the 114,688-token cap, 18 of 100 passed 30K tokens. The hard tier absorbed **1,382,425 of the run's 1,918,672 generation tokens — 72% of the work for 31% of the hard problems.** Same shape as Mirai, one-fifth the cost.

No effort knob exists on this model. The chat template defines neither `enable_thinking` nor `reasoning_effort`, and llama.cpp ignores both silently — no HTTP errors, unlike Mirai's invalid `high`. There is no bounded config to compare against. Full detail in [its model page](../models/lfm2.5-24b-a2b.md).

## Scoring

Two phases. Generation loops the model over every problem; scoring runs the official LiveCodeBench `codegen_metrics` function over generated code against hidden test cases in isolated subprocesses. Correctness follows the official rule: `all(result > 0)` per test case, where `-3` is an execution error, not a pass. Timeout-guarded per case.

**Harness:** `bench_lcb.py` (direct runner, official metric import, `--selftest` proves grading on known solutions before any scored run). The harness aborts rather than reporting a false `DONE` when zero problems produce code.

**Configs:**

| | Bounded | Unbounded |
|---|---|---|
| Reasoning effort | `medium` | `xhigh` (template max; `high` is invalid) |
| `--reasoning-budget` | 20,480 | *(removed)* |
| `--reasoning-max-tokens-floor` | 24,576 | *(removed)* |
| Harness `--max-tokens` | 32,768 | 114,688 |
| Harness `--req-timeout` | 1,200 | 7,200 |

Everything else identical in both runs: `-c 131072`, `-ngl 99`, `-fa on`, `-ctk/-ctv q8_0`, MTP speculative decoding (`--spec-draft-n-max 2`), refusal control vector at 1.0, temp 1.0 / top-p 0.95 / top-k 20.

**Hardware:** RTX 3090 24 GB, llama.cpp (Mirai S-ada fork), 300 W power cap.
