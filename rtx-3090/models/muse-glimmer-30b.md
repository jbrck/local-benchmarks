# Muse Glimmer 30B (GGUF Q4_K_M)

**Status:** tested | **Decision:** strong writer/generalist, weak coder. Best prose quality of any local 30B model we've tested.

| Spec | Value |
|---|---|
| Size | 30B (dense) |
| Quant | Q4_K_M (GGUF) |
| Engine | llama.cpp (build-muse-glimmer, CUDA sm81, F16) |
| Speed | 41 tok/s |
| VRAM | 16.5 GB (8 GB headroom at 32K ctx) |
| Architecture | `muse_glimmer` (requires recent llama.cpp with PR #26841) |
| Context | 32K tested (130K theoretical) |
| Chat format | reasoning_content split — outputs thinking in reasoning field, short answers in content |
| Flash-attention | DISABLED — slows Muse from 43 tok/s to 5 tok/s. The model's gated architecture conflicts with `-fa on` |

## Results

Muse Glimmer is a thinking model but produces useful content even within constrained budgets (unlike Nemotron which exhausts the ceiling on reasoning). Only 1/10 Prose ELO tasks hit the token cap with empty content.

| Test | Score | vs 27B field | Notes |
|---|---|---|---|
| MATH-500 | **82.6%** (413/500) | 5th of 13 | Consistent 82-83% throughout, near Nemotron |
| HumanEval+ | **62.2%** (102/164) | 11th of 13 | Weak — coding is not a strength |
| GPQA-Diamond | **56.6%** (112/198) | 2nd best (behind Holo4's 56.1% / tied OrcaSAQ2) | Strong graduate science, stable |
| Instruction v2 | **75.0%** (15/20) | 2nd best (behind swift's 19/20) | Excellent constraint following |
| Prose ELO | **1532**[¶](#fn-para) | 5th of 7 — pairwise ELO, on the same scale as the table |
| Tool-eval-bench | — | Not tested | |
| IFEval | — | Not tested with standard suite | |

## Analysis

### Strengths
- **Best writer of the 30B class** — 7.3/10 prose ELO with real content. TaglineSet, EmailReply, LinkedInPost, BlogIntro, PressRelease, CustomerTestimonial all scored 9/10.
- **GPQA 56.6%** is the best we've seen after correcting for partial results. Second best GPQA score overall.
- **8 GB VRAM headroom** is unmatched — almost as light as the 4B models but with 30B capability.
- **IFEval 75%** demonstrates solid instruction-following without needing special budgets.
- **Reasoning model that also outputs** — unlike Nemotron, Muse preserves enough token budget to produce visible content even at standard test limits.

### Weaknesses
- **Coding is poor** — HE+ at 62.2% is well below the Qwen3.8 variants (87-94%) and worse than both 4B models. If coding matters, skip this model.
- **Flash-attention must be off** — `-fa on` drops throughput from 43 tok/s to 5 tok/s. This limits context scaling efficiency.

### Notes
- The `build-muse` binary was inadequate — needed a rebuild with `-DGGML_CUDA=ON -DCUDA_ARCHITECTURES=81` to get usable speed.
- Flash-attention (`-fa on`) cuts throughput from 43 tok/s to 5 tok/s on this architecture. Must be disabled for Muse.
- With DFlash speculative decoding (draft GGUF available on HF), throughput could potentially reach 60-100 tok/s. Not tested.
- All battery processes used `setsid` + `PYTHONPATH=` to survive Hermes lifecycle events (SIGHUP isolation, Python 3.14 env pollution).
- The existing `Muse-Glimmer-30B` entry in the pruned table (1332 prose ELO) was from a different run — the full battery shows much better performance when served correctly.

## Timing

| Phase | Wall time |
|---|---|
| MATH-500 | ~3.5h |
| HumanEval+ | ~1h |
| GPQA-Diamond | ~1.5h |
| IFEval | ~5 min |
| Prose ELO | ~15 min |
| **Total** | **~6h** |