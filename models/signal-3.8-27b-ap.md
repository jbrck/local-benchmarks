# Signal-3.8-27B-AP [rejected]

**TL;DR:** Pruned. Mid on every axis; the token-efficiency marketing pitch didn't hold up (tool-eval 83, disclosed an injected payload in TC-57).

**Origin:** Base weights — **China** (Alibaba Qwen3.8-27B) · Publisher — `agentionali` (Hugging Face), publisher country not established

**Status:** pruned

## Tested on

| Hardware | GPU | CPU | RAM | OS | Setup details |
|---|---|---|---|---|---|
| [RTX 3090 24 GB](../rtx-3090/README.md) | NVIDIA GeForce RTX 3090 24 GB (power-limited to 300 W) | Intel Core i7-8700K, 6 cores / 12 threads | 64 GB | Ubuntu 24.04 | [rtx-3090 README](../rtx-3090/README.md#system) |

## Results

| Test | Score |
|---|---|
| MATH-500 | 85.6% |
| HumanEval+ | 84.1% |
| Tool-eval-bench | 83 (★★★★) — TC-57: disclosed injected payload |
| IFEval prompt/instruction | not run (rejected) |
| GPQA-Diamond | not run (rejected) |
| Instruction v2 | not run (rejected) |
| Reasoning bench | not run (rejected) |
| Refusal HARMFUL | not run (rejected) |

**Status:** pruned

## Timing

| Test | Total | Per item | Engine |
|---|---|---|---|
| MATH-500 (500 items) | *timing not captured* | *N/A* | llama.cpp GGUF |
| HumanEval+ (164 items) | **14min** | 5s | llama.cpp GGUF |

## Notes

Marketed as "token-efficient," but real writing and coding placed it solidly mid-pack. Coding 84.1% is a real notch below the qwen siblings (91.5%). Tool-eval showed a prompt-injection failure (TC-57: disclosed injected payload). Nothing it does better than the kept models, and some things worse. Pruned.