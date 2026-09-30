# Spark-X2.5-4B (GGUF Q4_K_M)

**Status:** tested | **Decision:** pending — strong coding/math for its size, but reasoning-content format breaks instruction-following bench scripts. Note: scores on GPQA, Instr v2, and similar tests are depressed by the model consuming its token budget on `reasoning_content` before producing an answer in `content` (the field the bench scripts read). True knowledge accuracy is higher than reported scores suggest.

| Spec | Value |
|---|---|
| Size | 4B parameters |
| Quant | Q4_K_M (GGUF) |
| Engine | llama.cpp upstream (b10828+, build `8df332de1`) |
| Speed | 185 tok/s at 64K ctx |
| VRAM | 6.3 GB (model + 64K KV) |
| Architecture | Hybrid attention — 1 full-attn + 3 sliding-window layers |
| Context | 1M native |
| Chat format | `spark2_5` / peg-native with `reasoning_content` separation |

## Results

| Test | Score | vs 4B field | Grade |
|---|---|---|---|---|
| MATH-500 | **72.2%** (361/500) | Strong for 4B | 🟢 |
| HumanEval+ | **70.7%** (116/164) | Strong for 4B | 🟢 |
| GPQA-Diamond | **25.3%** (50/198) | ⚠️ See note below | 🟡 |
| Instruction v2 | **0/20** (0%) | ❌ All empty — reasoning consumed budget | 🔴 |
| IFEval | **68.6% / 75.3%** | Below 27B field (77-80%) | 🟡 |
| Prose ELO | **1500** (4/10 with content) | ❌ 6/10 empty — reasoning consumed budget | 🔴 |
| Tool-eval-bench | **83 / ★★★★** | Strong for 4B, competitive with 27B agents | 🟢 |
| Refusal | Unknown | Format mismatch | ⚪ |
| Reasoning | 0/8 | ❌ All empty — reasoning consumed budget | 🔴 |

## Analysis

Spark-X2.5-4B is a **small, fast agentic model** that punches above its weight on coding and math but has a critical script-compatibility issue.

### Strengths
- **MATH 72.2%** — Competitive with 27B models (Holo4: 73.4%) at 1/6 the size
- **HE+ 70.7%** — Above any 2-3B model. Real coding capability in 4B parameters
- **Speed** — 185 tok/s on RTX 3090. Full battery ran in hours, not days
- **VRAM** — 6.3 GB leaves 18 GB free for other tasks
- **1M native context** — Can handle massive documents at this size
- **Apache 2.0 license** — No restrictions

### Weaknesses
- **Reasoning-content format mismatch** — The model outputs thinking in `reasoning_content` and only produces short answers in `content` after thinking completes. Bench scripts read `content` and score empty-as-fail. On GPQA, only 60/198 items produced content (83% accuracy when it did). Token budget (4096) is exhausted by thinking on complex queries before the answer appears.
- **Instr v2 0/20, Reasoning 0/8** — Not true zeros; the model never reached `content` output within script token limits
- **Refusal bench** — Format/parsing mismatch, no usable scores
- **4B knowledge ceiling** — Trained on less data than 27B models; GPQA 25.3% even accounting for the content issue is far below 27B field average (~45%)

### Context note
The `reasoning_content` field contains the model's full thinking, but the bench scripts only look at `content`. To get true scores, either (1) increase token limits so the model has room to think AND answer, or (2) modify scripts to fall back to `reasoning_content` when `content` is empty. For MATH and HE+, the model's thinking completes within the default budget — those scores are genuine.

## Comparison (vs 27B field — unfair but informative)

| vs | MATH | HE+ | GPQA | Verdict |
|---|---|---|---|---|
| qwen3.8-27b | −16.5% | −20.8% | −20.3% | 27Bs beat it everywhere — expected at 1/6 size |
| Holo4-27B | −1.2% | −19.5% | −30.8% | Closer on MATH than you'd expect |
| swift | −17.0% | −17.1% | −25.2% | 4B can't hang on coding or science |

## Verdict

Spark-X2.5-4B is **not competitive with 27B models** on raw knowledge or instruction-following, and the reasoning-content format issue makes it painful to benchmark. But at 6.3 GB VRAM / 185 tok/s with solid MATH and coding scores, it's a capable **low-power agentic sidekick** — perfect for embedding in n8n workflows, serving from an Ollama docker container, or running on a MacBook. If you need a model that writes functions and makes tool calls without burning server budget, this is a strong candidate. If you need PhD-level science or reliable instruction-following, look elsewhere.

Best use case: agentic middleware — write SQL, parse JSON, call APIs — on cheap hardware.