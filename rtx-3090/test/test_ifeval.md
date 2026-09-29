# IFEval (Instruction Following)

**What it evaluates:** 541 prompts, each with programmatically checkable constraints (length limits, forbidden words, format requirements, keyword frequency, case changes). Two scores: prompt-level (whole prompt followed) and instruction-level (individual constraints met — 834 total). No judge, no vibes — the checks are mechanical. Cheapest proxy for tool-call and format adherence.

**Reading scores:** Prompt >= 80% / instruction >= 85% is excellent locally. The qwen GGUF pair sits at 77/83. Sub-75 prompt means the model can't hold multi-part instructions. Weak spot for every local model: paragraph counting (one model scored 14.8% on that constraint).

## Results

| Model | Prompt | Instruction | Duration / tokens |
|---|---|---|---|
| nous-deepseek-v4-flash (remote) | 86.7% | 90.9% | |
| exl3-qwen3.8-27b § | 80.6% | 85.8% | always reasoning, higher token count |
| swift-qwen3.8-27b | 79.5% | 85.1% | 91 min, 223k tokens |
| GSQ-RCO-IQ3_S-mtp | 79.3% | 85.0% | 75 min, 216k tokens |
| qwen3.8-27b | 77.3% | 83.5% | |
| bonsai2 (PrismML tern PTQ1_0) | 74.3% | 81.8% | ~65 min, low reason tokens |
| ornith-1.5-35b-a3b (q4_k_s) | 75.0 / 82.7% | verbose coder |
| qwen3.8-27b-heretic | 77.1% | 83.3% | |
| crack2 (abliterated PQ2_0) | 77.3% | 84.2% | best IFEval of the abliterated set |
| qwen3-coder-30b | 74.3% | 82.0% | |
| OrcaSAQ-2-27B (vLLM) ‡ | 76.0% | 82.2% | 3.9h, 172k tokens — thinking disabled; 3x improvement over thinking-mode pilot 3.9h, 172k tokens — thinking disabled for IFEval; 3x improvement over thinking-mode pilot |

**Scoring:** tool-eval-bench's IFEval plugin checks 25 constraint types (length, word limits, JSON schema, forbidden words, start/end with, case transformation, etc.). Each prompt can carry multiple constraints; instruction-level measures whether each individual constraint was satisfied, prompt-level measures whether all constraints on a prompt were satisfied simultaneously.

**Hardware:** RTX 3090 24 GB, llama.cpp GGUF server (except EXL3 row), quantized models from HuggingFace.