# Easy 35-Task Smoke Battery

**What it evaluates:** 10 coding + 10 reasoning + 10 tool + 5 writing quick tasks, ~30 minutes per model. This was the screening pass that every candidate ran before earning a full battery. It ranks models one way; the full battery often flips it — gpt-oss scored 10/10 reasoning + 10/10 tool on smoke and looked like the champion, then finished dead last on the full battery.

**Smoke = floor check only. Full batteries = the verdict.**

**Reading scores:** A pass here means the model is worth testing for real. The coding column is unreliable for separation: the same 2 tasks (an empty-extraction quirk and a syntax error edge case) fail for EVERY model — treat "8/10 coding" as a wash across the board.

## Results

| Model | Coding | Reasoning | Tool |
|---|---|---|---|
| gpt-oss-20b-mxfp4 | 8/10 | 10/10 | 10/10 |
| qwen3.8-27b-heretic | 8/10 | 8/10 | 10/10 |
| qwen3.8-27b | 8/10 | 7/10 | 10/10 |
| Signal-3.8-27B-AP | 8/10 | 7/10 | 9/10 |
| Qwen-AgentWorld-35B-A3B | 8/10 | 4/10 | 7/10 |
| gemma-3-27b-it-qat | 8/10 | 5/10 | **0/10** |
| mistral-small-3.2-24b | 8/10 | 3/10 | 10/10 |

**Why the pruned models lost (from full battery data):**

- **gemma-3-27b-it-qat** — 0/10 tool use on smoke. Cannot call tools at all.
- **mistral-small-3.2-24b** — 3/10 reasoning on smoke. Too weak at multi-step.
- **Qwen-AgentWorld-35B-A3B** — 4/10 reasoning. Same problem.
- **gpt-oss-20b-mxfp4** — full battery dead last: MATH 74.2, HumanEval+ 67.7. Fails multi-step tool chains 12% of the time. Verbose writer (423-1800 tokens), unfixable by prompt. Smoke lied about it.
- **Signal-3.8-27B-AP** — mid on every axis; its "token-efficient" marketing pitch vanished on real prose.
- **Muse-Glimmer-30B** — bottom of prose ELO (1332): lost 9-1 to swift, tied qwen and GSQ 5-5, despite a newer architecture.
- **Twin-Turbo** — reasoning claims inverted (+44% thinking, 0.59x wall).
- **hermes-4.3-36b** — second-lowest prose ELO (1440, lost to qwen 7-3), mid math 81.0, HE+ 87.2. "Newer/bigger" did not beat the qwen pair.
- **qwen3-coder-30b-A3B** — "coder" that can't code: HE+ 73.2, toolbench 68 with a critical sleeper injection. Lost every battery axis.

**Hardware:** RTX 3090 24 GB, llama.cpp GGUF server.