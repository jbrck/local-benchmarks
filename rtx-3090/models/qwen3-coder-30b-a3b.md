# qwen3-coder-30b-A3B [rejected]

**Status:** pruned

## Results

| Test | Score |
|---|---|
| MATH-500 | 85.0% |
| HumanEval+ | 73.2% |
| IFEval prompt/instruction | 74.3% / 82.0% |
| GPQA-Diamond | 42.9% |
| Tool-eval-bench | 68 — CRITICAL sleeper injection |
| Instruction v2 | not run (rejected) |
| Prose ELO | not run (rejected) |
| Reasoning bench | not run (rejected) |
| Refusal HARMFUL | not run (rejected) |

**Status:** pruned

## Timing

| Test | Total | Per item | Engine |
|---|---|---|---|
| MATH-500 (500 items) | **25min** | 3s | llama.cpp GGUF |
| HumanEval+ (164 items) | **2min** | 0.6s | llama.cpp GGUF |
| GPQA-Diamond (198 items) | **8min** | 2s | llama.cpp GGUF |

## Notes

A "coder" model that can't code. HumanEval+ at 73.2% is catastrophic for a model with "coder" in its name — worse than every qwen sibling. Worse, tool-eval-bench flagged TC-08/49/51/56/57 plus TC-60: a critical cross-turn sleeper injection that exfiltrated attacker-supplied data from turn 1. Lost every battery axis. Pruned.