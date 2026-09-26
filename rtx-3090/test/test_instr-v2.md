# Instruction-Following v2 (Specialized Probe)

**What it evaluates:** 20 tests, ONE mechanical constraint each: exact word, word count, no-letter-e, JSON-to-schema, all-caps, starts-with, haiku-as-lines, and more. No judge. Pass/fail is unambiguous. Complements IFEval by isolating single-constraint obedience.

**Reading scores:** 19-20/20 is best-in-class locally. 18/20 is the qwen-family norm. Below that, a model can't be trusted with formatted outputs. Shared ceiling found on this hardware: no local model reliably obeys compound or artificial constraints (exactly-two-words answers come back as one word; no-letter-e fails at 8+ words). Treat that as a local-model ceiling, not a fixable defect.

## Results

| Model | Score |
|---|---|
| swift-qwen3.8-27b | 19/20 (95%) |
| qwen3.8-27b | 18/20 (90%) |
| bonsai2 (PrismML tern PTQ1_0) | 18/20 (90%) |
| ornith-1.5-35b-a3b (q4_k_s) | 19/20 (95%) | strong |
| qwen3.8-27b-heretic | 18/20 (90%) |
| crack2 (abliterated PQ2_0) | **19/20 (95%)** |
| GSQ-RCO-IQ3_S-mtp | 18/20 (90%) |

**Shared failure pattern:** the `two_words` test (reply with exactly two words) and the `no_e` test (write without the letter 'e' at 8+ words) fail for every local model. These are genuine capability ceilings, not model-specific defects.

**Hardware:** RTX 3090 24 GB, llama.cpp GGUF server.