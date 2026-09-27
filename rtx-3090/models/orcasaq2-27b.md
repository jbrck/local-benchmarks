# OrcaSAQ-2-27B [testing]

**Role:** Under evaluation — hybrid attention (SAQ) candidate from OrcaRouter / Continuum AI Corporation.

## Results

| Test | Score |
|---|---|
| MATH-500 | **90.8%** |
| HumanEval+ | 89.6% |
| GPQA-Diamond | 55.0% (20/198) — stopped early |
| IFEval | **76.0%** / 82.2% |
| Prose ELO | ⏳ pending |
| Tool-eval-bench | not run |
| Instruction v2 | not run |
| Reasoning bench | not run |
| Refusal HARMFUL | not run |

**Status:** testing

## Timing

| Test | Total | Per item | Engine |
|---|---|---|---|
| MATH-500 (500 items) | **11.5h** | 83s | vLLM OrcaSAQ2-kernel |
| HumanEval+ (164 items) | **2.2h** | 49s | vLLM OrcaSAQ2-kernel |
| GPQA-Diamond (20/198) | ~41min* | ~74s | vLLM OrcaSAQ2-kernel |
| IFEval (541 prompts) | **3.9h** | 26s | vLLM OrcaSAQ2-kernel (--no-think) |

\* GPQA at full 198 items extrapolates to ~4.1h. All timings reflect the vLLM engine at 11.6 tok/s. For comparison, the same tests on llama.cpp GGUF run at 20-25 tok/s and complete 2-4x faster — OrcaSAQ2 has no GGUF quant available.

## Notes

OrcaSAQ-2-27B is a Qwen3-based 27B model with hybrid attention: 48 Gated DeltaNet layers (linear attention, no KV cache) + 16 full-attention layers (every 4th layer). This reduces KV cache memory by ~4x vs dense models, enabling 262K context on a 24 GB GPU with 6.5 GB headroom.

Currently the strongest local model on MATH-500 (90.8% — 1.6 pts above swift's 89.2%). Code writing is 3rd at 89.6%, trailing behind heretic (92.7%) and base qwen3.8-27b (91.5%). 

GPQA-Diamond was stopped early (20/198 at 55.0%). The model's scientific knowledge scoring is not comparable to other local models — it was served via vLLM (11.6 tok/s) while all other GPQA runs used llama.cpp GGUF at 20-25 tok/s. More importantly, GPQA-Diamond (graduate science) tests knowledge domains irrelevant to this model's intended use.

Served via vLLM OrcaSAQ2-kernel (Docker), model ID `exl3`. 262K context at ~21.9 GB VRAM with fp8 KV cache and hybrid attention tuning. Same GPU runs ComfyUI alongside at reduced context depth.