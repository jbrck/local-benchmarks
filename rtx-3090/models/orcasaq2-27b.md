# OrcaSAQ-2-27B [kept]

**TL;DR:** Math + long-context specialist on vLLM. MATH 90.8% (2nd best local). GPQA 43.9% on a full 198-item run (Sep 18 orcarouter stack). OrcaSAQ2 kernel only.

**Role:** Math leader, 262K context — hybrid attention (SAQ) model from OrcaRouter / Continuum AI Corporation.

## Results

| Test | Score |
|---|---|
| MATH-500 | **90.8%** |
| HumanEval+ | 89.6% |
| GPQA-Diamond | 43.9%[†](#fn-stuck) (87/198, full run) — orcarouter stack; later kernel-stack runs broke |
| IFEval | **76.0%** / 82.2% [‡](#fn-dagger) |
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

\* GPQA at full 198 items extrapolates to ~4.1h. All timings reflect the vLLM engine at 11.6 tok/s. For comparison, the same tests on llama.cpp GGUF run at 20-25 tok/s and complete 2-4x faster — OrcaSAQ2 has no GGUF quant available.

## Notes

OrcaSAQ-2-27B is a Qwen3-based 27B model with hybrid attention: 48 Gated DeltaNet layers (linear attention, no KV cache) + 16 full-attention layers (every 4th layer). This reduces KV cache memory by ~4x vs dense models, enabling 262K context on a 24 GB GPU with 6.5 GB headroom.

Currently the strongest local model on MATH-500 (90.8% — 1.6 pts above swift's 89.2%). Code writing is 3rd at 89.6%, trailing behind heretic (92.7%) and base qwen3.8-27b (91.5%). 

GPQA-Diamond has three runs on record. The Sep 18 orcarouter-stack full run scored **43.9% (87/198)** — that's the number I stand behind. A 20-item partial (55.0%) was abandoned before JSON saved; it's unverified and superseded. Two later attempts on the OrcaSAQ2-kernel stack (26.8%, 5.6%) show classic serving breakage — self-feedback thinking loops consuming the token budget, null responses — not model capability. The model's scientific knowledge scoring is also not directly comparable: vLLM at 11.6 tok/s vs llama.cpp GGUF at 20-25 tok/s for every other local model.

Served via vLLM OrcaSAQ2-kernel (Docker), model ID `exl3`. 262K context at ~21.9 GB VRAM with fp8 KV cache and hybrid attention tuning. Same GPU runs ComfyUI alongside at reduced context depth.

## Reproduction notes

This battery was harder than usual. Specific failures and their resolutions:

**GPQA-Diamond (1024 → 4096 → three runs).** Default 1024 max_tokens produced 71% null responses — the model spent the entire budget on `[think]...[/think]` and vLLM stripped the markers, leaving empty content. Reran at 8192 (later 4096) to match the test standard. The Sep 18 full run on the orcarouter stack is the recorded score (43.9%). Two later OrcaSAQ2-kernel attempts broke on serving, not capability. **Fix for reproducers:** set `--max-tokens 4096` minimum, and decide upfront whether vLLM vs llama.cpp comparison is meaningful for your question.

**IFEval (stuck → pilot → --no-think).** First attempt ran without max_tokens or --no-think. The model entered thinking mode and generated reasoning indefinitely — first request appeared stuck for 47 minutes. A 3-prompt pilot revealed the fix: thinking mode scored 33% in 4m51s, while `--no-think` scored 100% in 2m47s (3x faster, 3x more accurate). Full run used `--no-think --no-live` and completed 541 prompts in 3.9h at 76.0/82.2%. **Fix for reproducers:** always pilot 3 IFEval prompts with vs without `--no-think` before committing to a full run on thinking models.


---

<a id="fn-dagger">‡</a> `--no-think` mode.
<a id="fn-stuck">†</a> Stuck-loop — model entered a self-referential reasoning loop; run stopped early.
<a id="fn-caret">^</a> LiteLLM proxy — served through the LiteLLM proxy instead of direct inference.
