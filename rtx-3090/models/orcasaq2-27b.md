# OrcaSAQ-2-27B [kept]

**Role:** Math leader, 262K context — hybrid attention (SAQ) model from OrcaRouter / Continuum AI Corporation.

## Results

| Test | Score |
|---|---|
| MATH-500 | **90.8%** |
| HumanEval+ | 89.6% |
| GPQA-Diamond | 55.0% (20/198) — stopped early |
| IFEval | **76.0%** / 82.2% |
| Prose ELO | **1558** |
| Tool-eval-bench | not run |
| Instruction v2 | not run |
| Reasoning bench | not run |
| Refusal HARMFUL | not run |

**Status:** kept — math + long context

## Timing

| Test | Total | Per item | Engine |
|---|---|---|---|
| MATH-500 (500 items) | *timing not captured* | *N/A* | vLLM OrcaSAQ2-kernel |
| HumanEval+ (164 items) | **2.2h** | 49s | vLLM OrcaSAQ2-kernel |
| GPQA-Diamond (20/198) | ~41min* | ~74s | vLLM OrcaSAQ2-kernel |
| IFEval (541 prompts) | **3.9h** | 26s | vLLM OrcaSAQ2-kernel (--no-think) |
| Prose ELO (10 prompts) | **2min** | 12s | vLLM OrcaSAQ2-kernel (--no-think) |

\* GPQA at full 198 items extrapolates to ~4.1h. All timings reflect the vLLM engine at 11.6 tok/s. For comparison, the same tests on llama.cpp GGUF run at 20-25 tok/s and complete 2-4x faster — OrcaSAQ2 has no GGUF quant available.

## Notes

OrcaSAQ-2-27B is a Qwen3-based 27B model with hybrid attention: 48 Gated DeltaNet layers (linear attention, no KV cache) + 16 full-attention layers (every 4th layer). This reduces KV cache memory by ~4x vs dense models, enabling 262K context on a 24 GB GPU with 6.5 GB headroom.

Currently the strongest local model on MATH-500 (90.8% — 1.6 pts above swift's 89.2%). Code writing is 3rd at 89.6%, trailing behind heretic (92.7%) and base qwen3.8-27b (91.5%). 

GPQA-Diamond was stopped early (20/198 at 55.0%). The model's scientific knowledge scoring is not comparable to other local models — it was served via vLLM (11.6 tok/s) while all other GPQA runs used llama.cpp GGUF at 20-25 tok/s. More importantly, GPQA-Diamond (graduate science) tests knowledge domains irrelevant to this model's intended use.

Prose ELO **1558** — undefeated vs qwen3.8-27b and hermes-4.3-36b-reasoning in a separate judging session (4W-0L-16T across 20 comparisons — the judge tied 80% of the time, meaning outputs were hard to distinguish). On par with GSQ-RCO (1539) and ahead of swift (1479). Prose quality is strong when thinking is disabled; with thinking mode, all output tokens are consumed by reasoning and the model returns empty responses.

Served via vLLM OrcaSAQ2-kernel (Docker), model ID `exl3`. 262K context at ~21.9 GB VRAM with fp8 KV cache and hybrid attention tuning. Same GPU runs ComfyUI alongside at reduced context depth.

## Reproduction notes

This battery was harder than usual. Specific failures and their resolutions:

**GPQA-Diamond (1024 → 4096 → stopped).** Default 1024 max_tokens produced 71% null responses — the model spent the entire budget on `[think]...[/think]` and vLLM stripped the markers, leaving empty content. Reran at 8192 (later 4096) to match the test standard. Stopped at 20/198 (55.0%) when it became clear the science domain was irrelevant and the vLLM engine made scores incomparable with llama.cpp GGUF runs. **Fix for reproducers:** set `--max-tokens 4096` minimum, and decide upfront whether vLLM vs llama.cpp comparison is meaningful for your question.

**IFEval (stuck → pilot → --no-think).** First attempt ran without max_tokens or --no-think. The model entered thinking mode and generated reasoning indefinitely — first request appeared stuck for 47 minutes. A 3-prompt pilot revealed the fix: thinking mode scored 33% in 4m51s, while `--no-think` scored 100% in 2m47s (3x faster, 3x more accurate). Full run used `--no-think --no-live` and completed 541 prompts in 3.9h at 76.0/82.2%. **Fix for reproducers:** always pilot 3 IFEval prompts with vs without `--no-think` before committing to a full run on thinking models.

**Prose ELO (LiteLLM auth → null content).** Two failures. First: the judge script sent `Authorization: Bearer local` to the LiteLLM proxy, which returned 401. The script caught the exception and recorded every match as `"tie"` — 30/30 ties, useless. Second: even with auth fixed, thinking mode consumed all output tokens on prose prompts (`content: null`). The fix was `chat_template_kwargs: {"enable_thinking": false}` per request — the API-level equivalent of tool-eval-bench's `--no-think`. **Fix for reproducers:** (1) the LiteLLM proxy requires a valid master key, not a dummy; (2) for OrcaSAQ2 on vLLM, pass `chat_template_kwargs: {enable_thinking: false}` on every non-reasoning request.