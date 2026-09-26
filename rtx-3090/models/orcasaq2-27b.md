# OrcaSAQ-2-27B [testing]

**Role:** Under evaluation — hybrid attention (SAQ) candidate from OrcaRouter / Continuum AI Corporation.

## Results

| Test | Score |
|---|---|
| MATH-500 | **90.8%** |
| HumanEval+ | 89.6% |
| GPQA-Diamond | ⏳ rerunning (70.0% at 10/198) |
| IFEval | ⏳ running |
| Prose ELO | ⏳ pending |

**Status:** testing

## Notes

OrcaSAQ-2-27B is a Qwen3-based 27B model with hybrid attention: 48 Gated DeltaNet layers (linear attention, no KV cache) + 16 full-attention layers (every 4th layer). This reduces KV cache memory by ~4x vs dense models, enabling 262K context on a 24 GB GPU with 6.5 GB headroom.

Currently the strongest local model on MATH-500 (90.8% — 1.6 pts above swift's 89.2%). Code writing is 3rd at 89.6%, trailing behind heretic (92.7%) and base qwen3.8-27b (91.5%). GPQA and IFEval results pending.

Served via vLLM OrcaSAQ2-kernel (Docker), model ID `exl3`. 262K context at ~21.9 GB VRAM with fp8 KV cache and hybrid attention tuning. Same GPU runs ComfyUI alongside at reduced context depth.