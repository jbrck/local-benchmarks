# GPQA-Diamond

**What it evaluates:** 198 graduate-level science multiple-choice questions (biology, chemistry, physics). Expert knowledge + careful reasoning. Random guessing scores 25%. This is the hardest knowledge test in the battery.

**Reading scores:** 44-48% is the 27B-class norm on this hardware. 50%+ is exceptional. 40-43% is below par. Frontier models reach 60-80%. The remote flash baseline's 83.8% shows how far local knowledge still is from cloud — the gap is bigger here than on any other test.

## Results

| Model | Accuracy | Notes |
|---|---|---|
| nous-deepseek-v4-flash | 83.8% | remote baseline |
| swift-qwen3.8-27b | 50.5% | best local; 136 average tokens (half of the next closest) |
| GSQ-RCO-IQ3_S-mtp | 48.0% | |
| exl3-qwen3.8-27b | 46.5% | |
| qwen3.8-27b | 46.0% | |
| bonsai2 (PrismML tern PTQ1_0) | 43.9% | |
| ornith-1.5-35b-a3b (q4_k_s) | **34.3%** | worst local; knowledge gap |
| qwen3.8-27b-heretic | 44.4% | 1.8x qwen's tokens, lower score |
| crack2 (abliterated PQ2_0) | 43.4% | |
| qwen3-coder-30b | 42.9% | |

**Scoring:** The model receives a free-form question with (A)-(D) options and is instructed to reply with `\boxed{letter}`. The parser extracts the LAST `\boxed{letter}` in the output (reasoning may contain boxed references; the final answer comes at the end).

**Token budget:** Qwen3-based models (including the bonsai and swift variants) output reasoning inside `[think]...[/think]` tags before the answer. With a 1024-token cap, the model spends the entire budget on thinking and produces zero visible content — this affected 71% of responses in one early run. All GPQA figures here use a minimum 4096-token cap (8192 recommended for these architectures).

**Hardware:** RTX 3090 24 GB, llama.cpp GGUF server (except EXL3 row), quantized models from HuggingFace.