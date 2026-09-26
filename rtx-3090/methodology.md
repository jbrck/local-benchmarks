# Methodology — How these benchmarks are run

The results here were produced by automating the full test lifecycle through an AI agent (Hermes Agent by Nous Research) running on a local workstation. This document explains the automation layer — the harnesses, the orchestration, and how each test gets its numbers. The goal is to make results reproducible and transparent.

## Overview

The test pipeline works in stages:

1. **Smoke screening** — every candidate model runs a quick 35-task battery (~30 minutes). Models that pass proceed to the full battery.
2. **Full battery** — MATH-500 + HumanEval+ (the primary verdict), then GPQA-Diamond and IFEval as secondary axes, then tool-eval-bench for agentic capability.
3. **Specialized probes** — prose ELO (writing quality), instruction-following v2 (mechanical constraint obedience), refusal battery (safety alignment), reasoning bench (thinking-token efficiency).
4. **Serving comparisons** — speculative decode A/B (MTP vs DFlash2), context ceiling, engine comparisons (llama.cpp GGUF vs exllamav3).

Each stage runs unattended — the agent loads a model, runs the harness, saves results, swaps to the next model, and repeats.

## How the agent orchestrates tests

The agent has access to the workstation's terminal and filesystem. A session begins with a goal ("benchmark these 4 models against the full battery"), and the agent works through it step by step:

- **Model selection** — the agent picks a model from the swap list, stops any running model, loads the new one, and waits for the server to report healthy.
- **Harness invocation** — for each test, the agent calls a Python harness script with the correct endpoint URL and output directory. The harness handles all API calls and score calculation.
- **Progress monitoring** — harnesses stream progress to log files that the agent polls. The agent reports checkpoints: "MATH complete, starting HumanEval+".
- **Result collection** — each harness writes a JSON results file. The agent reads it, records the scores in a tracking table, and either proceeds to the next model or the next test.
- **Aggregation** — when all models finish a battery, the agent produces a markdown report with the summary table, per-test breakdowns, and token usage.

The key design choice: **benchmarks run as independent scripts, not through eval harness wrappers.** The agent controls the flow, but each test is its own focused Python script. This avoids coupling to any particular evaluation framework and makes adding a new test straightforward — write a Python script that takes an endpoint URL and writes a results JSON, then add it to the agent's task list.

## The direct-runner pattern

Every harness in this battery follows the same shape:

```python
# Pseudocode for the common pattern
def main():
    endpoint = os.environ.get("BENCH_API_BASE", "http://127.0.0.1:8080/v1")
    model    = os.environ.get("BENCH_MODEL_ID", "model")
    api_key  = os.environ.get("BENCH_API_KEY", "local")
    out_dir  = os.environ.get("BENCH_OUT_DIR", "./results")

    # Load dataset (from HuggingFace or bundled JSON)
    items = load_dataset()

    # Run each item through the endpoint
    results = []
    for i, item in enumerate(items):
        response = chat_completion(endpoint, model, item.prompt)
        score = evaluate(item, response)
        results.append(score)
        if i % 10 == 0:
            write_progress(out_dir, model, i, len(items))

    # Calculate final scores
    summary = aggregate(results)
    write_results(out_dir, model, summary, results)
```

Each harness reads `BENCH_API_BASE` so it can target any OpenAI-compatible endpoint — local GGUF server, exllamav3 server, or a remote API proxy. The same code tests all three.

## Model swapping

With a single 24 GB GPU, only one model fits in VRAM at a time. The swap cycle:

1. Save the current model's active configuration
2. Stop the inference server (releases ~22 GB VRAM)
3. Load the new model's GGUF file
4. Verify the server reports healthy on `/health`
5. Run the battery

A swap takes roughly 1-2 minutes depending on model size (5-21 GB files). The agent tracks swap history to avoid cycling back to a model that was already tested.

## What each harness does

| Test | Harness | Scoring method |
|---|---|---|
| MATH-500 | Direct runner | `math_verify` (exact answer match from `\boxed{...}`) |
| HumanEval+ | Direct runner | evalplus test suite — generated code executed against hidden tests with a 45-second timeout |
| GPQA-Diamond | Direct runner | Last `\boxed{letter}` in output (free-form response, 4-choice questions) |
| IFEval | tool-eval-bench plugin | 25 mechanical constraint types checked programmatically — no judge |
| Tool-eval-bench | tool-eval-bench full runner | 69 multi-turn agentic scenarios, per-scenario pass/partial/fail with safety warnings |
| Prose ELO | Writing generator + neutral judge | Pairwise comparison by a frontier model (nous-deepseek-v4-flash), anonymized prompts, randomized side assignment, ELO k=32 |
| Instruction v2 | Direct runner | 20 single-constraint mechanical checks — exact word, word count, no-letter-e, JSON schema, etc. |
| Refusal battery | Direct runner | Pattern-based REFUSE/COMPLY classification across 70 prompts in three tiers |
| Reasoning bench | Direct runner | Thinking-token estimation from `reasoning_content` field, wall time, correctness |

## Models tested

Links to the exact model files (or their base repos) tested on this system. Several of these are community fine-tunes and merges with GGUFs produced by third-party quantizers — the link points to the best place to find the quant used unless the base weights were the tested artifact. Models marked "pruned" were removed from the active stack after the full battery showed deficits that smoke screening missed.

### Kept models

| Model | Source / HF link | Notes |
|---|---|---|
| qwen3.8-27b (GGUF Q5_K_M) | [ggml-org/Qwen3.8-27B-GGUF](https://huggingface.co/ggml-org/Qwen3.8-27B-GGUF) | Official GGUF of the base Qwen3.8-27B. Baseline reference. |
| qwen3.8-27b-heretic | [heretic-org/Qwen3.8-27B-heretic-ara](https://huggingface.co/heretic-org/Qwen3.8-27B-heretic-ara) | Abliterated (uncensored) variant. GGUF quants via community quantizers. Writing/fiction only — not for untrusted inputs. |
| swift-qwen3.8-27b | Community merge on Qwen3.8-27B by flabcat/swift-group. GGUF via bartowski or similar. | Best math (89.2%) and GPQA (50.5%) on this hardware. Safest refusal profile (100% harmful). Default model. |
| GSQ-RCO-IQ3_S-mtp | [ISTA-DASLab/Qwen3.8-27B-GSQ-RCO-GGUF](https://huggingface.co/ISTA-DASLab/Qwen3.8-27B-GSQ-RCO-GGUF) | ISTA-DASLab per-tensor mixed-precision quant. Task-lossless at 3.5 bpw (11.8 GB). MTP head bundled. Best VRAM footprint. |
| bonsai2 (tern PTQ1_0) | [prism-ml/Ternary-Bonsai-2-27B-gguf](https://huggingface.co/prism-ml/Ternary-Bonsai-2-27B-gguf) | PrismML ternary-quantized model. 5.9 GB, needs fork of llama.cpp (prism branch). 128K+ context at ~10 GB VRAM. |
| orcarouter (LoRA 2.0) | [prism-ml/Ternary-Bonsai-LoRA-2.0](https://huggingface.co/prism-ml/Ternary-Bonsai-LoRA-2.0) | Runtime LoRA applied atop the ternary bonsai PTQ1_0 base. Same prism fork required. |
| ornith-1.5-35b-a3b (q4_k_s) | [mudler/Ornith-1.5-35B-A3B-APEX-MTP-GGUF](https://huggingface.co/mudler/Ornith-1.5-35B-A3B-APEX-MTP-GGUF) | 35B MoE, 21 GB file, 64K max context on the 3090. Best local coding (94.5% HE+). Worst GPQA (34.3%). |
| crack2 (abliterated PQ2_0) | [prism-ml/CRACK-2-PQ2_0](https://huggingface.co/prism-ml/CRACK) | PrismML weight-abliterated variant. PQ2_0 packing (7.2 GB). HE+ collapsed to 76.2% from abliteration. |
| exl3-qwen3.8-27b (3.5bpw) | same weights as qwen3.8-27b above, served via [exllamav3](https://github.com/turboderp/exllamav3) | Same weights, different serving engine. Always reasons. |

### Pruned models (with rationale and links)

| Model | Source / HF link | Why pruned |
|---|---|---|
| gpt-oss-20b-mxfp4 | [openai/gpt-oss-20b](https://huggingface.co/openai/gpt-oss-20b) | Dead last on full battery: MATH 74.2%, HE+ 67.7%, tool 75, 12% multi-step chains. Smoke misled. |
| hermes-4.3-36b | [NousResearch/Hermes-4.3-36B](https://huggingface.co/NousResearch/Hermes-4.3-36B) | 2nd-lowest prose ELO (1440). Mid math (81.0%) and HE+ (87.2%). Bigger does not beat the Qwen pair. |
| qwen3-coder-30b-A3B | [Qwen/Qwen3-Coder-30B-A3B-Instruct](https://huggingface.co/Qwen/Qwen3-Coder-30B-A3B-Instruct) | "Coder" that can't code: HE+ 73.2%, toolbench 68 with critical sleeper injection. Lost every axis. |
| gemma-3-27b-it-qat | [google/gemma-3-27b-it-qat-gguf](https://huggingface.co/google/gemma-3-27b-it-qat-gguf) | 0/10 tool calls on smoke — GGUF lacks function-call template support. Disqualified. |
| mistral-small-3.2-24b | [mistralai/Mistral-Small-3.2-24B-Instruct-2509](https://huggingface.co/mistralai/Mistral-Small-3.2-24B-Instruct-2509) | 3/10 reasoning on smoke. Too weak at multi-step. |
| Qwen-AgentWorld-35B-A3B | [Qwen/Qwen-AgentWorld-35B-A3B](https://huggingface.co/Qwen/Qwen-AgentWorld-35B-A3B) | 4/10 reasoning on smoke. Same problem. |
| Signal-3.8-27B-AP | Community merge. | Mid on every axis. Token-efficiency pitch didn't survive real prose. |
| Muse-Glimmer-30B | [Muse-AI/Muse-Glimmer-30B](https://huggingface.co/Muse-AI/Muse-Glimmer-30B) | Bottom of prose ELO (1332). Lost 9-1 to swift. |
| Twin-Turbo | Community merge. | Reasoning claims inverted: +44% thinking, 0.59x wall speed. |

## What happens after the runs

The agent creates a master results document with:
- A summary table (every model × every test)
- Per-test deep dives with methodology, score interpretation, and caveats
- Token usage tables (total and per-model per test)
- Per-model verdicts (keep or prune, with rationale)

The master doc lives in the internal notes and is the single source of truth. These public pages are derived from it.

## Reproducing

Same hardware, same engine (llama.cpp), same quantized model files (available on HuggingFace). The harnesses are all open-source or published:

- MATH-500 dataset: HuggingFace (standard, not gated)
- HumanEval+ via evalplus package
- GPQA-Diamond: `hendrydong/gpqa_diamond` on HuggingFace (not gated)
- tool-eval-bench: [github.com/SeraphimSerapis/tool-eval-bench](https://github.com/SeraphimSerapis/tool-eval-bench) (includes IFEval plugin)
- Prose ELO: custom judge script using the same frontier model

All harnesses accept `BENCH_API_BASE` to target your own endpoint. Model files cited in each test's notes are the specific quantizations tested — exact filenames on HuggingFace.

## Special considerations

Several things we learned the hard way running these tests. Worth knowing if you try to reproduce:

### Reasoning models eat token budgets

Qwen3-family models think by default. They output reasoning inside `[think]...[/think]` tags before the visible answer. If the output token cap is too low (e.g. 1024), the model spends the entire budget thinking and the response comes back empty with `finish_reason: "length"`. This affected 71% of GPQA responses in one early run and caused the first EXL3 HumanEval+ run to score 74.4% instead of 87.8%.

**Fix:** Turn reasoning off per-model (`LLAMA_ARG_REASONING=off` in llama.cpp server config) for tests where it's not needed. For tests that require reasoning (GPQA, reasoning bench), set `max_tokens` to at least 4096 (8192 recommended).

### EXL3 can't disable reasoning

The exllamav3 serving engine always reasons — there is no off switch. This means every response burns output tokens on chain-of-thought before reaching the answer. Every test on EXL3 needs a correspondingly higher token budget. The HumanEval+ correction (1024 → 4096 cap recovered +13.4 points) shows how large the penalty is.

### ComfyUI steals VRAM

When ComfyUI (image generation) is loaded, it holds approximately 11 GB of VRAM even when idle. A 24 GB card effectively has ~13 GB free for LLM work. A model that should fit (e.g. a 19 GB GGUF) will fail to load with a `cudaMalloc failed` error that looks like a quant problem but is actually VRAM contention.

**Fix:** Free ComfyUI's models before swapping — an API call releases ~10.6 GB without stopping the service. Check `nvidia-smi --query-compute-apps` before any unattended run.

### Generated code can contain infinite loops

HumanEval+ executes model-generated Python code against hidden tests. Models can produce valid Python that enters a `while True` loop. One model hung the full battery for 2 hours before the break was noticed.

**Fix:** All code execution is timeout-guarded with a 45-second SIGALRM. Your harness needs the same guard.

### tool-eval-bench `--backend` is an enum, not free-form

The flag accepts: `llamacpp`, `litellm`, `vllm`, `sglang`, `gemini`, `ninfer`. Using `--backend openai` produces a result JSON with `final_score: null` — it completes in ~8 seconds with no errors and the null score file looks like a successful run. Trip up: the skip-if-file-exists logic then treats the null artifact as complete.

**Fix:** Use `--backend llamacpp` for a local GGUF server, `--backend litellm` for a proxy, `--backend vllm` for any plain OpenAI-wire server.

### Prose ELO judge routing is deceptive

The prose ELO judge script hardcodes the local endpoint. Passing a remote judge name (like `nous-deepseek-v4-flash` via `--judge`) is silently ignored — llama.cpp ignores the `model` field and serves whatever GGUF is loaded. The judge label in old result files is misleading.

**Fix:** Use a separate script that routes through the model proxy and anonymizes the prompt (sample A/B naming instead of model names). Also randomize side assignment to cancel position bias.

### Smoke tests are not verdicts

The 35-task smoke battery ranks models one way; the full battery often flips it. gpt-oss scored 10/10 reasoning + 10/10 tool on smoke and looked like the champion. The full battery showed it dead last: MATH 74.2%, HE+ 67.7%, 12% on multi-step tool chains. **Smoke = floor check only. Full batteries = the verdict.**

### Community merge card claims are unreliable in both directions

Swift's card claimed 1.95x speedup (measured 1.41x). Twin-Turbo (DavidAU) claimed 1/2-to-1/20 thinking tokens (measured +44% MORE thinking, 0.59x wall time). Marketing numbers are directional at best and massaged for hype. Always verify with the reasoning bench before assigning a model to a role based on its card.

### DFlash2 ceiling claim didn't reproduce

A tweet claimed DFlash2 loads ~240k context vs MTP's ~200k on this model and GPU. Both loaded the full native 262k without issue, and the ceiling test pushed both past 400k (MTP: 400k, DFlash2: 420k). The "240k vs 200k" claim was wrong on this box. Verify context ceiling with real load+completion tests, not tweet numbers.