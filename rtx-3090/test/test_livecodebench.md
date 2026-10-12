# LiveCodeBench

**TL;DR:** 100 competition coding problems with hidden test cases, no contamination window. Mirai S Qwen3.8-27B scored **66.0%** overall — but 78.6% of the problems it actually solved. It loses 16 points to non-termination: on hard problems it thinks past the output budget and never writes code.

**What it evaluates:** Competition programming problems drawn from LeetCode, AtCoder, and Codeforces, each with hidden test cases executed against the generated solution. Problems are date-stamped, so results can be filtered to a contamination-free window. This is a harder coding test than HumanEval+ — algorithmic thinking on unseen problems, not boilerplate completion.

**Reading scores:** 60-70% is solid for a 27B on hard problems. The easier split (LeetCode medium, AtCoder easy) is where a capable model lives; the hard tail separates models. Watch the no-code rate — a model that reasons well but doesn't terminate scores low for the wrong reason.

## Results

| Model | Accuracy | Notes |
|---|---|---|
| **Mirai S Qwen3.8-27B** (2.4 bpw trellis) | **66.0%** (66/100) | thinking on; 84/100 produced code, 16 failed to terminate |

## Breakdown

**By platform:**

| Platform | Score | No-code failures |
|---|---|---|
| leetcode | 25/32 = 78% | 2 |
| atcoder | 41/68 = 60% | 14 |

**By difficulty:**

| Difficulty | Score |
|---|---|
| easy | 24/24 = 100% |
| medium | 26/31 = 84% |
| hard | 16/45 = 36% |

The sample skews hard (45 of 100 are hard). Easy and medium are near-solved; the damage is concentrated in the hard tier and on AtCoder, where 14 of 16 non-terminations occurred.

## Where the 16 failures come from

The model's normal pattern: think in the reasoning channel (~20K tokens, the server's `--reasoning-budget` ceiling), then emit a short content block with the code. On ~25% of hard problems the budget fires, the server pushes it to answer — and it keeps analyzing, in the content channel. That channel has no budget, so it burns the remaining output on prose and never writes code.

Evidence from the capped responses:

| Group | Content channel | Reasoning channel |
|---|---|---|
| Solved (with code) | 548–3,298 chars | 21K–74K chars |
| Capped (no code) | **35K–43K chars** | 56K–74K chars |

The tails of the capped responses are mid-sentence analysis, not partial code. A larger output budget buys more analysis, not a solution — 25 problems at ~64 tok/s would add hours and likely still miss.

**Scorable pass rate:** of the 84 problems that produced code, 66 passed = 78.6%. That's the model's true coding capability on this set. The 66.0% headline is what you get when a quarter of hard problems never reach a code block.

## Speed

| Metric | Value |
|---|---|
| Items | 100 |
| Overall pass | 66 (66.0%) |
| Scorable pass | 66/84 (78.6%) |
| Wall clock | 21,195.7 s (5.89 h) |
| Generation tokens | 1,490,818 |
| Throughput | 70.3 tok/s |
| Median tokens/problem | 12,617 |

## Scoring

Two phases. Generation loops the model over every problem; scoring runs the official LiveCodeBench `codegen_metrics` function over generated code against hidden test cases in isolated subprocesses. Correctness follows the official rule: `all(result > 0)` per test case, where `-3` is an execution error, not a pass. Timeout-guarded per case.

**Harness:** `bench_lcb.py` (direct runner, official metric import, `--selftest` proves grading on known solutions before any scored run).

**Hardware:** RTX 3090 24 GB, llama.cpp (Mirai S-ada fork), q8_0 KV, MTP speculative decoding, 300 W power cap.
