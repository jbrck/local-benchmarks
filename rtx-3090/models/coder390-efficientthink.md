# Qwen3.8-27B-Coder390-EfficientThink

**TL;DR:** Thinking-tuned Qwen3.8 that needs 30K+ reasoning tokens per GPQA question — unusable pace at Q3 on 24 GB (projected 50+ hr). GPQA 87.1% thinking-on (n=70 partial) vs 48.0% off. HE+ 79.9% is worst of the kept field. Pruned.

| Property | Detail |
|---|---|
| Source | [nerkyor](https://huggingface.co/nerkyor/Qwen3.8-27B-Coder390-EfficientThink-Opus5.5-GPT6Astra-Grok4.7-DSV4Pro-K3-SFT-RLOO-MTP-DFlash2) |
| Base | Qwen3.8-27B, SFT + SimPO + 2x RLOO rounds |
| Format | GGUF Q3LynnStyle-Q8MTP (17 GB) |
| Engine | llama.cpp (port 11434), 131K ctx, q8_0 KV |
| Quant penalty | Author's own GGUF tables show -2 to -4 pts at Q8_0/Q6_K vs FP8; Q3 likely worse |
| License | Apache 2.0 |

## Battery results

| Test | Score | Notes |
|---|---|---|
| MATH-500 | **89.0%** | 4th locally — behind Mirai S (94.2), OrcaSAQ (90.8), swift (89.2) |
| HumanEval+ | **79.9%** | Worst of the kept 27B field (87-95%). The "coder" branding did not transfer |
| GPQA-D (thinking off) | **48.0%** | Comparable row. Model needs to think; starved it and it scored like a base model |
| GPQA-D (thinking on) | **87.1%** (n=70, partial) | Best-shot row. See caveats |
| IFEval | **71.5% prompt / 84.9% instr** | Mid-pack. 44/541 timeouts at the 120s client limit |
| Instr v2 | **20/20** | First perfect score on this probe locally |

Not run: Tool-eval. Instr v2: **20/20** — first model to post a perfect score on that probe.

## The GPQA story — read before citing either number

The author's tweet claims **GPQA 89.9%**. The model card backs it to 178/198 (89.9%) — but that's static FP8 on two RTX PRO 6000s with a 94,208-token thinking budget on a 100K window. Their own GGUF table shows Q8_0 at 171/198 (86.4%) and Q6_K at 174/198 (87.9%). Quantization already costs 2-4 points on their own hardware.

Our measurement: thinking disabled, the model scored **48.0%** — near the base-model floor. This is a model RL-trained to think long and stop naturally; answering cold cripples it. With thinking enabled at a 32K cap, it hit **87.1% through 70 items** (run killed, no saved JSON — reconstructed from the killed run's live log). That matches the author's own Q6_K/Q8_0 GGUF numbers.

The full 94K-budget rerun was abandoned at ~11 items: the model averaged **~31K thinking tokens per question** at Q3-quant decode speed (~31 tok/s), projecting **50+ hours** for 198 items. The 87.1% figure is from n=70 at a 32K cap — below the author's P90 of ~28K, so truncation may cost it a point or two, but the sample size is solid.

**Translation:** the 89.9% claim is real on their hardware, my quant gets close to their own GGUF numbers, but only if you let it think — and on a 24 GB card, letting it think costs hours.

## Verdict

Interesting model, wrong tool for this box. Its EfficientThink training works as advertised (94K truncations drop 4→1 on their GPQA runs), and thinking-on GPQA is genuinely strong. But:

- **"Coder" is a misnomer** — HumanEval+ 79.9% is bottom of the kept field. The LCB 90 claim may be real, but LCB is not HE+, and on the test I actually run it loses to every qwen sibling.
- **Thinking-on runs are not viable at Q3 on a 3090.** 31 tok/s × 31K-token thinking chains = 17 minutes per question. A 27B model that needs 30K tokens to answer GPQA is a luxury for 2x-RTX-6000 rigs.
- **Thinking-off behavior is broken by design** — 48% GPQA. If your serving stack can't do 30K-token generations, don't bother with this model.

Pruned. The qwen3.8-27B family (vanilla / swift / Mirai S) remains the local standard.
