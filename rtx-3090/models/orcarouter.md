# orcarouter (PTQ1_0 + LoRA 2.0) [kept]

**Role:** Uncensored, runtime-swappable LoRA steering.

## Results

| Test | Score |
|---|---|
| MATH-500 | 85.4% |
| HumanEval+ | 87.8% |
| IFEval prompt/instruction | 74.7% / 82.5% |
| GPQA-Diamond | 43.9% |
| Tool-eval-bench | 78 (★★★) — safety-capped |
| Instruction v2 | 17/20 |
| Refusal HARMFUL | 0/20 (0%) — refused none |
| Refusal HARMFUL note | no working code produced |

**Status:** kept, uncensored

## Notes

Bonsai2 base (5.9 GB ternary) plus a 9 MB runtime LoRA that steers behavior. The LoRA makes it uncensored — 0% refusal on all three tiers. However, its "compliant" harmful responses did not produce working criminal code (unlike heretic), and its tool-eval score dropped to 78 with safety caps triggered. MATH and HE+ are near-identical to bonsai2. The runtime LoRA is a unique capability — swap LoRA files without reloading the base model — but the uncensored output plus agentic weakness makes it a narrow-use tool.