# Instruction-Following v2 (Specialized Probe)

**What it evaluates:** 20 tests, ONE mechanical constraint each: exact word, word count, no-letter-e, JSON-to-schema, all-caps, starts-with, haiku-as-lines, and more. No judge. Pass/fail is unambiguous. Complements IFEval by isolating single-constraint obedience.

**Reading scores:** 19-20/20 is best-in-class locally. 18/20 is the qwen-family norm. Below that, a model can't be trusted with formatted outputs. Shared ceiling found on this hardware: no local model reliably obeys compound or artificial constraints (exactly-two-words answers come back as one word; no-letter-e fails at 8+ words). Treat that as a local-model ceiling, not a fixable defect.

## Results

| Model | Score | Notes |
|---|---|---|
| swift-qwen3.8-27b | 19/20 (95%) | |
| qwen3.8-27b | 18/20 (90%) | |
| bonsai2 (PrismML tern PTQ1_0) | 18/20 (90%) | |
| ornith-1.5-35b-a3b (q4_k_s) | 19/20 (95%) | strong |
| qwen3.8-27b-heretic | 18/20 (90%) | |
| crack2 (abliterated PQ2_0) | **19/20 (95%)** | |
| GSQ-RCO-IQ3_S-mtp | 18/20 (90%) | |
| **Muse Glimmer 30B** | **15/20 (75%)** [¶](#fn-para) | Good — clears constraints on most tasks. 3 letter-e failures, 1 two-words failure, 1 exact-word failure |
| **Nemotron Cascade 2 30B A3B** | **9/20 (45%)** [¶](#fn-para) | Depressed — burned budget on reasoning, left content empty or incomplete on 11 tasks |

**Shared failure pattern:** the `two_words` test (reply with exactly two words) and the `no_e` test (write without the letter 'e' at 8+ words) fail for every local model. These are genuine capability ceilings, not model-specific defects.

**Hardware:** RTX 3090 24 GB, llama.cpp GGUF server.

---

<a id="fn-para">¶</a> Reasoning-content format mismatch — model outputs full thinking in `reasoning_content` and only short answers in `content`. At standard token budgets (400-1500), the model burns most tokens on reasoning. Scores reflect the constrained budget, not the model's ceiling.