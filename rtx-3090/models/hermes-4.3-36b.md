# hermes-4.3-36b [rejected]

**Status:** pruned

## Results

| Test | Score |
|---|---|
| MATH-500 | 81.0% |
| HumanEval+ | 87.2% |
| Prose ELO | 1440 |
| IFEval prompt/instruction | not run (rejected) |
| GPQA-Diamond | not run (rejected) |
| Tool-eval-bench | not run (rejected) |
| Instruction v2 | not run (rejected) |
| Reasoning bench | not run (rejected) |
| Refusal HARMFUL | not run (rejected) |

**Status:** pruned

## Timing

| Test | Total | Per item | Engine |
|---|---|---|---|
| MATH-500 (500 items) | **2.1h** | 15s | llama.cpp GGUF |
| HumanEval+ (164 items) | **9min** | 3s | llama.cpp GGUF |

## Notes

A 36B model that couldn't beat the qwen 27B pair on any axis. MATH (81.0) is well behind the qwen sibling norm (85-89%). HumanEval+ (87.2) is mid-pack. Prose ELO (1440) is second-lowest — lost 7-3 to vanilla qwen. "Newer/bigger" did not beat the qwen family on this hardware. Pruned.