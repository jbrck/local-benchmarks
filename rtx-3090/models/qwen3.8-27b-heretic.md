# qwen3.8-27b-heretic [kept — writing only]

**Role:** Writing/fiction only. Do not expose to untrusted input.

## Results

| Test | Score |
|---|---|
| MATH-500 | 85.8% |
| HumanEval+ | 92.7% |
| IFEval prompt/instruction | 77.1% / 83.3% |
| GPQA-Diamond | 44.4% |
| Tool-eval-bench | 92 (★★★★★) |
| Instruction v2 | 18/20 |
| Prose ELO | 1631 |
| Reasoning bench | not run |
| Refusal HARMFUL | 2/20 (10%) — working criminal code |

**Status:** kept, writing-only

## Timing

| Test | Total | Per item | Engine |
|---|---|---|---|
| MATH-500 (500 items) | **4.0h** | 29s | llama.cpp GGUF |
| HumanEval+ (164 items) | **19min** | 7s | llama.cpp GGUF |
| GPQA-Diamond (198 items) | **1.4h** | 26s | llama.cpp GGUF |

## Notes

Best local writer by prose ELO (1631 — head of the field). Scored 92 on tool-eval-bench with no safety warnings. But the refusal battery is a hard red line: 0% harmful refusal, and several "compliant" responses contained working credit-card stealer code, doxxer code, and ransomware. Never use this model agentically or with untrusted input. Writing generation only, and only in a sandboxed context.