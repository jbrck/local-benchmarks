# gpt-oss-20b-mxfp4 [rejected]

**Status:** pruned

## Results

| Test | Score |
|---|---|
| MATH-500 | 74.2% |
| HumanEval+ | 67.7% |
| Tool-eval-bench | 75 (★★★★) — TC-48: sent update to unintended recipient |
| Smoke reasoning | 10/10 |
| Smoke tool | 10/10 |
| IFEval prompt/instruction | not run (rejected) |
| GPQA-Diamond | not run (rejected) |
| Instruction v2 | not run (rejected) |
| Prose ELO | not run (rejected) |
| Reasoning bench | not run (rejected) |
| Refusal HARMFUL | not run (rejected) |

**Status:** pruned

## Timing

| Test | Total | Per item | Engine |
|---|---|---|---|
| MATH-500 (500 items) | **34min** | 4s | llama.cpp GGUF |
| HumanEval+ (164 items) | **6min** | 2s | llama.cpp GGUF |

## Notes

The poster child for why smoke tests aren't verdicts. Scored a perfect 10/10 on smoke reasoning and 10/10 on smoke tool — looked like the local champion. The full battery told a different story: dead last on MATH (74.2), dead last on HumanEval+ (67.7), multi-step tool chain failure at 12%. Also a verbose writer (423-1800 tokens per response) that no system prompt could fix. The smoke battery ranked it first; the full battery ranked it last. Pruned.