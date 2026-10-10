# hermes-4.3-36b [rejected]

**TL;DR:** Pruned. 36B lost to the 27B qwen pair on every axis (MATH 81.0, HE+ 87.2). Bigger did not beat better.

**Status:** pruned

## Results

| Test | Score |
|---|---|
| MATH-500 | 81.0% |
| HumanEval+ | 87.2% |
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
| MATH-500 (500 items) | *timing not captured* | *N/A* | llama.cpp GGUF |
| HumanEval+ (164 items) | **9min** | 3s | llama.cpp GGUF |

## Notes

A 36B model that couldn't beat the qwen 27B pair on any axis. MATH (81.0) is well behind the qwen sibling norm (85-89%). HumanEval+ (87.2) is mid-pack. "Newer/bigger" did not beat the qwen family on this hardware. Pruned.