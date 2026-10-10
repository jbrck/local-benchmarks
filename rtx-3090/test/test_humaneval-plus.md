# HumanEval+

**TL;DR:** 164 Python coding problems against hidden tests. Local best: Mirai S at 95.1%. Code scores are the most quant-sensitive leg — sub-80 on a 27B model means quant or serving damage.

**What it evaluates:** 164 Python programming problems (HumanEval with the evalplus extended test suite — more tests than the original). Generated code is executed against hidden tests, not eyeballed. Measures code generation that actually runs.

Token budget matters: reasoning models spend part of their output budget on chain-of-thought before writing code. A 1024-token output cap is enough for non-reasoning models (code is short) but truncates reasoning models mid-code. All reasoning-model figures here use a 4096-token cap; the difference is visible in the EXL3 correction below.

**Reading scores:** 91-93% is the local ceiling. 87-88% is good but a real notch down. Sub-80% means frequent broken code.

## Results

| Model | Accuracy | Notes |
|---|---|---|
| nous-deepseek-v4-flash | 93.9% | remote baseline |
| qwen3.8-27b-heretic | 92.7% | best local |
| bonsai2 (PrismML tern PTQ1_0) | 89.0% | tiny VRAM; mid coding |
| **OrcaSAQ-2-27B** (vLLM) | 89.6% | SAQ-trained; 3rd, behind heretic & base qwen |
| ornith-1.5-35b-a3b (q4_k_s) | **94.5%** | best local coding |
| qwen3.8-27b | 91.5% | |
| crack2 (abliterated PQ2_0) | **76.2%** | severe coding degrade from abliteration |
| GSQ-RCO-IQ3_S-mtp | 91.5% | 11.8 GB file, full-size score |
| swift-qwen3.8-27b | 87.8% | best math, mid coding |
| exl3-qwen3.8-27b (3.5bpw) [§](#fn-section) | 87.8% | at 4096 cap |
| hermes-4.3-36b | 87.2% | |
| Signal-3.8-27B-AP | 84.1% | |
| qwen3-coder-30b | 73.2% | |
| gpt-oss-20b-mxfp4 | 67.7% | |

**The EXL3 HumanEval+ correction.** The first EXL3 run scored 74.4%. Root cause: EXL3 cannot disable reasoning, and the harness capped responses at 1024 tokens — the model spent the budget thinking and the code got truncated (39/164 items hit the cap; 1/39 passed). GGUF runs had reasoning off, so all 1024 tokens went to code. Re-run at a 4096-token cap: **87.8%** (144/164), average 997 tokens, 16 items still exhausted the cap. So +13.4 points was artifact; the remaining ~4-5 point gap to the GGUF pair is genuine — coding is the one axis where GGUF Q5_K_M keeps the edge over EXL3 3.5bpw. Reasoning models need a token budget or they eat it all thinking; this pattern repeats across engines.

**Scoring:** Generated code is saved to a `.py` file, then evalplus runs it against the extended test suite. Execution is timeout-guarded at 45 seconds per item (generated code can contain infinite loops).


---

<a id="fn-section">§</a> Reasoning forced — model architecture cannot disable thinking.

**Hardware:** RTX 3090 24 GB, llama.cpp GGUF server (except EXL3 row, which used exllamav3), quantized models from HuggingFace.