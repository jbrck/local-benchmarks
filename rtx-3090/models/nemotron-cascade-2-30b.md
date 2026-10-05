# Nemotron Cascade 2 30B A3B (GGUF Q3_K_M)

**Status:** tested | **Decision:** fast, thinking model that needs above-budget token allocations to shine.

| Spec | Value |
|---|---|
| Size | 30B total, 3B active (A3B fine-grained MoE) |
| Quant | Q3_K_M (GGUF) |
| Engine | llama.cpp (standard build) |
| Speed | 140-186 tok/s |
| VRAM | 19.9 GB (4 GB headroom at 32K ctx) |
| Architecture | Nemotron (MoE, 3B/30B cascade) |
| Context | 32K tested (128K theoretical) |
| Chat format | reasoning_content split — outputs thinking in reasoning field, short answers in content |

## Results

Nemotron is a **thinking model** — it writes full CoT in `reasoning_content` and leaves `content` near-empty on constrained budgets. MATH and HE+ extractors parse reasoning, so those scores are genuine. IFEval, Prose ELO, and GPQA at standard budgets are depressed because the model exhausts its token ceiling on reasoning.

| Test | Score | vs 27B field | Notes |
|---|---|---|---|
| MATH-500 | **81.2%** (406/500) | 6th of 12 | Below Qwen3.8 variants, above Holo4 and 4B models |
| HumanEval+ | **84.8%** (139/164) | 8th of 12 | Respectable, trails Qwen3.8 variants but above crack2 |
| GPQA-Diamond | **54.0%** (107/198) | 3rd best (tied with Holo4/OrcaSAQ2) | Strong for graduate science |
| Instruction v2 | **45.0%** (9/20) | Depressed by reasoning budget | Same mechanism as Spark-X2.5-4B |
| Prose ELO | **2.5/10** | Depressed — 8/10 tasks produced empty content | Same mechanism |
| Tool-eval-bench | — | Not tested | |
| IFEval | — | Not tested with standard suite | |

## Analysis

### Strengths
- **Fastest model in its size class** — 140-186 tok/s is ~3x Qwen3.8 variants. If token budgets were adjusted proportionally, Nemotron would score competitively across the board.
- **MATH 81.2%** is solid for a 3B-active MoE — only 7 pts behind the 27B leader.
- **GPQA 54%** places it in the top 3 local models for graduate science knowledge.
- **4 GB VRAM headroom** on a 24 GB card leaves room for ComfyUI (11 GB) leaving ~10 GB for inference.

### Weaknesses
- **Thinking model overhead** — every request burns 400-1500 tokens on reasoning before producing visible output. This penalizes it on standard-budget benchmarks designed for non-thinking models.
- **Prose quality collapses** on 1500-token budget — 8/10 tasks produce blank `content` fields. With 4x budget it would likely be competitive.
- **IFEval 45%** — constraint-following is poor at 400-token budget. The model cannot produce a simple two-word answer within that limit.

### What would help
- Rerun IFEval and Prose ELO with 4-8x the default budget to match the model's thinking overhead
- Compare against other reasoning/thinking models (OrcaSAQ-2-27B, Spark) at budget-adjusted settings
- Test with Bonsai 2's speculative decode for potential speed gains