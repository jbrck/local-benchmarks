# Instruction-Following v2 (Specialized Probe)

**TL;DR:** 20 single-constraint obedience tests, no judge. Local ceiling: ~19-20/20 for qwen-family models. Coder390 is the only perfect 20/20. Below 15/20 means untrustworthy formatted output.

**What it evaluates:** 20 tests, ONE mechanical constraint each: exact word, word count, no-letter-e, JSON-to-schema, all-caps, starts-with, haiku-as-lines, and more. No judge. Pass/fail is unambiguous. Complements IFEval by isolating single-constraint obedience.

**Reading scores:** 19-20/20 is best-in-class locally. 18/20 is the qwen-family norm. Below that, a model can't be trusted with formatted outputs. Shared ceiling found on this hardware: no local model reliably obeys compound or artificial constraints (exactly-two-words answers come back as one word; no-letter-e fails at 8+ words). Treat that as a local-model ceiling, not a fixable defect.

## Results

| Model | [Origin](../README.md#results-summary) | Score | Notes |
|---|---|---|---|
| swift-qwen3.8-27b | China | 19/20 (95%) | |
| qwen3.8-27b | China | 18/20 (90%) | |
| bonsai2 (PrismML tern PTQ1_0) | China | 18/20 (90%) | |
| ornith-1.5-35b-a3b (q4_k_s) | not est. | 19/20 (95%) | strong |
| qwen3.8-27b-heretic | China | 18/20 (90%) | |
| crack2 (abliterated PQ2_0) | China | **19/20 (95%)** | |
| GSQ-RCO-IQ3_S-mtp | China (AT pub.) | 18/20 (90%) | |
| **Nanbeige4.3-33B** (Q4_K_S) | China | 17/20 (85%) | |
| **Muse Glimmer 30B** | USA | **15/20 (75%)** [¶](#fn-para) | Good — clears constraints on most tasks. 3 letter-e failures, 1 two-words failure, 1 exact-word failure |
| **Nemotron Cascade 2 30B A3B** | USA | **9/20 (45%)** [¶](#fn-para) | Depressed — burned budget on reasoning, left content empty or incomplete on 11 tasks |
| **Mirai S Qwen3.8-27B** (2.4 bpw trellis) | China | **20/20 (100%)** | |
| **Coder390-EfficientThink** (Q3LynnStyle-Q8MTP) | China | **20/20 (100%)** | first perfect score; Mirai S also posts 20/20 |
| Qwen3.8-27B-TurboFCFusion ("turbo-fable", Q4_K_M) | China | **19/20 (95%)** | |
| **Nemotron 3.5 Bolt 74B-A9B** (IQ2_XXS, 9B active) | USA | **19/20 (95%)** | |
| Holo4-27B (Q4_K_M, Q8 KV) | China (FR pub.) | 9/20 (45%) | thinking ate the budget |

**Shared failure pattern:** the `two_words` test (reply with exactly two words) and the `no_e` test (write without the letter 'e' at 8+ words) fail for every local model. These are genuine capability ceilings, not model-specific defects.

**Hardware:** RTX 3090 24 GB, llama.cpp GGUF server.

---

<a id="fn-para">¶</a> Reasoning-content format mismatch — model outputs full thinking in `reasoning_content` and only short answers in `content`. At standard token budgets (400-1500), the model burns most tokens on reasoning. Scores reflect the constrained budget, not the model's ceiling.