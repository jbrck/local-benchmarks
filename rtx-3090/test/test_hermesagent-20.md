# HermesAgent-20 (Agentic Harness Behavior)

**TL;DR:** 20 scenarios that drive a real agent runtime inside a Docker verifier and grade the artifacts it produces. Where tool-eval-bench scores whether a model can call a tool, this scores whether it can run a loop — write memory, patch a skill, schedule a job, handle an approval prompt, delegate, and recover a context. Score is per-scenario 0–1.

**What it evaluates:** Twenty scenarios (HA-01 … HA-20) covering memory read/write, skill creation and patching, cron scheduling, approval handling, delegation, artifact export, and context recovery. The sandbox boots a pinned Hermes Agent runtime, gives it a task, and captures what it actually wrote to disk. Scoring reads the runtime artifacts deterministically — no judge model, no LLM grading. A scenario either produced the right artifact or it didn't.

**Agentic capability, not prose.** The leg is deliberately adjacent to tool-eval-bench and does not replace it. Tool-eval-bench answers "can the model emit the right tool call." This answers "does the model drive a runtime to a correct end state, and how many tokens does it burn getting there." The second question is the one that decides whether a model is usable as an agent brain on a 24 GB card.

**Reading scores:** `score` is pass rate over the 20 scenarios. Runaway count must be read alongside it — a model that scores well by never terminating is not a good agent. Token spend is the third number; two models at the same score are not the same model if one spent 4× the tokens.

**Reading the result JSON:** Read the score from `packs[0].pass_at_k.score`. Read thinking state from `packs[0].thinking_enabled`. **Never read the top-level `thinking_enabled` field** — it echoes the absent CLI override and reads `false` even when thinking is on. Token totals live at `tokens.{prompt,completion,total}`; the top-level `extra_body` is `null` even when reasoning is enabled, so do not use it to infer sampling.

## Results

| Model | Origin | Score | Scenarios | Runaway | Tokens | Wall | Notes |
|---|---|---|---|---|---|---|---|
| Tess-4.5-27B (Q6_K) | China | 1.0 on 2 of 20 | 2 | 0 | 3,676 / 26,168 | 20.9 s / 114.8 s | partial — HA-01 and HA-02 only, integration smoke |
| Nanbeige4.3-33B (Q4_K_S) | China | 15/20 (75%) pass@k | 20 | 0 | 554,338 | 56.8 min | strict-totals grading 12/20; thinking disabled |

Blank means not run. The backfill plan lives in `hermes/docs/benchmarks/hermesagent-20-backfill-plan.md`.

**Scoring detail:** per-scenario artifact score plus a harvest/artifact tier. `failure_mode` names the reason on a miss. Percentages and token totals are stored per scenario, so a per-scenario burn rate is recoverable from the JSON.

## Running it

Prerequisites on Badger-001, already satisfied:

```bash
# benchlocal-cli with the sandbox extra
/mnt/data/benchmark-venv/bin/benchlocal-cli --version
# sandbox image, Hermes agent baked at commit 44cdf55
docker images benchlocal-sandbox-hermes:latest
```

Free ComfyUI's VRAM first, then run against any OpenAI-compatible endpoint:

```bash
curl -X POST http://127.0.0.1:8188/free -d '{"unload_models":true,"free_memory":true}'

/mnt/data/benchmark-venv/bin/benchlocal-cli run \
  --pack hermesagent-20 --enable-sandboxed-packs \
  --endpoint http://127.0.0.1:11434/v1 --model <model-id> \
  --timeout-per-case 600 --incremental \
  --output json --save-json <out>/<model>-hermesagent20.json
```

`--enable-sandboxed-packs` is mandatory. Without it the pack is a stub and the scenarios dispatch to nothing — the run looks clean and scores nothing.

`--incremental` writes `<save-json>.partial.jsonl` as each scenario completes. A clean finish folds the partial into the final JSON and deletes the sidecar; a crash leaves it resumable:

```bash
benchlocal-cli run --resume <save-json>.partial.jsonl ...
```

Wall-clock budget: budget on failure, not on success. `--timeout-per-case 600` bounds each scenario at 10 minutes. Worst case for the full pack is ~3.3 h if every scenario times out; observed full scenarios are far faster.

**Wall clock and tok/s are mandatory for this leg** — the whole point is finding models that return useful agent behavior without burning a token bonfire.

## Pitfalls

- **Top-level `thinking_enabled` lies.** It is a CLI-override echo. Resolved thinking state comes from the pack defaults (`default_thinking: on` for this pack) and shows up at `packs[].thinking_enabled` and `sampling_params.chat_template_kwargs.enable_thinking`. Verified `true` on a live run.
- **Sampling is greedy here, the model card is not.** The harness resolves `temperature: 0, top_p: 1, max_tokens: 16384`. Tess's published card specifies `temperature: 1.0, top_p: 0.95, top_k: 20`. A score produced under card sampling is **not** comparable to a battery row.
- **The verifier is a real agent runtime, so an unpinned agent is a confound.** BenchLocal's own release notes admit an earlier unpinned-agent harness artifact. The image here bakes the agent at commit `44cdf55`; re-baking on a different commit changes the harness, not just the model.
- **Do not run this leg against a single-slot server that is already serving.** Queueing behind another client reads as a runaway model. Check for in-flight work before starting.
- **`/health` is not a readiness gate.** It returns `ok` on a wedged slot.

**Source:** [stevibe/HermesAgent-20](https://github.com/stevibe/HermesAgent-20) via [benchlocal-cli](https://github.com/stevibe/benchlocal-cli) — commit `4a9c017`, sandbox image `benchlocal-sandbox-hermes:latest`.