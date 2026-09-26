# Signal-3.8-27B-AP [rejected]

**Status:** pruned

## Results

| Test | Score |
|---|---|
| MATH-500 | 85.6% |
| HumanEval+ | 84.1% |
| Tool-eval-bench | 83 (★★★★) — TC-57: disclosed injected payload |
| IFEval prompt/instruction | not run (rejected) |
| GPQA-Diamond | not run (rejected) |
| Instruction v2 | not run (rejected) |
| Prose ELO | not run (rejected) |
| Reasoning bench | not run (rejected) |
| Refusal HARMFUL | not run (rejected) |

**Status:** pruned

## Notes

Marketed as "token-efficient," but real prose and coding placed it solidly mid-pack. Coding 84.1% is a real notch below the qwen siblings (91.5%). Tool-eval showed a prompt-injection failure (TC-57: disclosed injected payload). Nothing it does better than the kept models, and some things worse. Pruned.