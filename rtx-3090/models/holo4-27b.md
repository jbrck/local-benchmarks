# Holo4-27B (GGUF Q4_K_M)

**Status:** tested | **Decision:** pending — strong GPQA and coding, weak math and IFEval. Neither prunable nor a clear keeper.

| Spec | Value |
|---|---|
| Size | 27B parameters |
| Quant | Q4_K_M (GGUF) |
| Engine | llama.cpp (prismML build), Q8 KV cache at 131K ctx |
| Speed | 23.1 tok/s at 131K ctx (Q8 KV) |
| VRAM | 22.6 GB |
| Architecture | DBRX-style (fine-grained MoE w/ intermediate size 7 slots) |
| Context | 262K native (121K usable at Q8 KV) |
| Chat format | peg-native |

## Results

| Test | Score | vs 27B field | Grade |
|---|---|---|---|
| MATH-500 | **73.4%** (367/500) | 2nd-lowest (only gpt-oss-20b is worse) | 🔴 |
| HumanEval+ | **90.2%** (148/164) | 3rd-best (ornith 94.5%, heretic 92.7%) | 🟢 |
| GPQA-Diamond | **56.1%** (111/198) | **Best local score** (OrcaSAQ2 55.0% partial) | 🟢 |
| Instruction v2 | **45.0%** (9/20) | Below average | 🔴 |
| IFEval Prompt | **71.5%** | Below average | 🔴 |
| IFEval Instruction | **77.6%** | Below average (swift 85.1%, EXL3 85.8%) | 🔴 |
| Tool-eval-bench | **91** | Tied with EXL3 (OrcaSAQ2 none, heretic 92) | 🟢 |
| Prose ELO | 1500 (baseline) | Default — no reference model for pairwise | ⚪ |
| Refusal H | **20/20 (100%)** | Perfect | 🟢 |
| Refusal B | **2/30 (7%)** | Slight over-refusal, normal | 🟢 |
| Reasoning | **7/8 pass** | Passable | 🟢 |

## Analysis

Holo4 is a **split personality**. It leads the field on GPQA (graduate science) and competes for top spot on coding and tool use, but collapses on basic mechanical constraints (IFEval, Instruction v2) and lags on math.

### Strengths
- **HE+ 90.2%** — Nearly matches ornith (94.5%) and heretic (92.7%) without sacrificing quality
- **GPQA 56.1%** — The only local model to break 55% on graduate science. Suggests strong knowledge retention
- **Tool-eval 91** — Tied with EXL3 for second place. Solid agentic capability

### Weaknesses
- **MATH 73.4%** — 15 points below the field average. For a reasoning-heavy model this is surprising
- **IFEval 77.6% I** — Below every Qwen3 variant. Struggles with mechanical constraints (paragraph counting, letter frequency)
- **Instr v2 45.0%** — Nearly half of simple single-constraint tests failed. Pattern suggests the model doesn't handle negative constraints well
- **Prose ELO** — No meaningful score yet. 4/10 outputs hit max length (1500 tok), suggesting verbose tendency

### Context note
The initial battery run (262K FP16 KV cache, 3.2 tok/s) produced 16.6% MATH — an artifact of context corruption from the 340 GB KV cache overflowing to system RAM. All scores above are from the **Q8 KV cache run** at 131K context, which stabilized at 23.1 tok/s and produced valid results.

## Comparison

| vs | MATH | HE+ | GPQA | IFEval I | TE | Verdict |
|---|---|---|---|---|---|---|
| qwen3.8-27b | −13.4% | −1.3% | **+10.1%** | −5.9% | +3 | Trade math for GPQA/tools |
| swift | −15.8% | +2.4% | +5.6% | −7.5% | +2 | Swift beats on math but lags on coding |
| OrcaSAQ2 | −17.4% | +0.6% | — | −4.6% | — | Orca dominates math |
| ornith | −11.8% | −4.3% | +21.8% | −5.1% | +3 | Trade best coding for best science |

## Verdict

Holo4 is a **specialist, not a generalist**. It's the best local model for graduate-level science reasoning (GPQA) and competitive on coding and tool use, but its mechanical constraint-following and math scores are weak enough that it can't replace a Qwen3 variant as a daily driver.

Not a keeper for general use. Worth keeping as a **GPQA specialist** if you need science-heavy agentic work.