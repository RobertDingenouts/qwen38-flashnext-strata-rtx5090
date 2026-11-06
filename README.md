# Qwen3.8-Flash-Next 177B MoE — daily driver on one RTX 5090

**TL;DR** — A 177B-parameter MoE language model (125B MoE backbone + 51B PLE n-gram tables) runs as a *daily assistant* on one RTX 5090 (32 GB VRAM) with 63 GB system RAM: **262K context, ~260 t/s decode at long context, vision included**, plus music/image/TTS pipelines co-resident on the same GPU. The winning quant is Unsloth Dynamic **UD-IQ3_XXS + DFlash2 drafting**; the exact failure boundary of 63 GB RAM is measured, not guessed. What you need: RTX 50-series (sm_120), ≥64 GB RAM, ≥100 GB fast NVMe, Strata ≥0.2.11 with engine ≥0.1.57.

*Speeds are my own measurements on this exact box; recipes are reproducible down to checksums — including what failed.*

---

## Hardware

| Component | Spec |
|---|---|
| GPU | NVIDIA RTX 5090 (32 GB VRAM, sm_120) |
| CPU | Intel Core Ultra 9 285K |
| RAM | 63 GB DDR5 |
| Storage | NVMe Gen4 (~5 GB/s effective) |
| OS | Ubuntu, kernel 7.0, CUDA 13.3 |

## Quickstart (5 steps)

1. Install [Strata](https://github.com/Niko1221/Strata) ≥ v0.2.11 and download the CUDA-13 engine package 0.1.57.
2. Download `unsloth/Qwen3.8-Flash-Next-GGUF` → quant **UD-IQ3_XXS** (pinned revision `d96201194f103c906994f283f504209dc8d2a10c`; ~84 GB of files on disk) plus `mmproj-BF16.gguf` for vision.
3. Build the DFlash2 drafter pack: clone `incoai/Qwen3.8-Flash-Next-DFlash2` (revision `a22a1744e85c17364dcc54a6a420af7135d4c7f6`), run Strata's `tools/dflash2_pack.py --q4_0` on `model.safetensors` → expect pack SHA-256 `cda40d62d0eb8f37ed6e272142641a1d2f0de4a32f9e14d0416e7a71534849f1` (Strata's own pinned hash, bit-for-bit). Optionally build the smaller `-selq4` variant too (`--q4_0 --q4_0-selector`, SHA-256 `4a42b98220ea7880664002c8edda1563aecbae754b762a2e2f31b2b91ee93ecd`); the engine auto-prefers it. Gotcha: Strata's drafting module only activates DFlash2 when the **plain** `q4_0.pack` exists on disk — keep both files.
4. Start with `configs/strata-ud-iq3.example.json` (fill in your own API key). Key engine flags: `--max-context 262144 --kv int8 --kv-resident 32768 --mmap-experts --expert-cache auto --prefill auto --draft dflash2 --draft-model /path/to/dflash2/qwen3.8-flash-next-dflash2.q4_0-selq4.pack --draft-fallback none --spec 6`, env `STRATA_HC_Q8=1`, `STRATA_ROUTE_TAIL_SKIP=0`.
5. Verify: query `/v1/models`, send one long prompt, and check the engine log for `--draft dflash2` on the command line plus steady `drafts accepted` ratios (~75%).

## Measured performance (2026-11-06, temperature 0, 2 runs per point)

Decode t/s by context length — median of the two runs; **both raw samples are in `results/`**, so min/max per point is readable there:

| context | IQ3_S GSQ-RCO (MTP) | UD-IQ4_XS (DFlash2) | **UD-IQ3_XXS (DFlash2)** |
|---|---|---|---|
| 1K   | **222** | 35 | 189 |
| 4K   | **236** | 64 | 198 |
| 8K   | **276** | 81 | 189 |
| 32K  | **282** | 160 | 251 |
| 131K | 232 | 62 | **277** |
| 262K | 203 | 211 | **261** |

Draft acceptance: IQ3_S/MTP 71% · UD-IQ3_XXS/DFlash2 76% · UD-IQ4_XS/DFlash2 79%.
Prefill plateau on engine 0.1.57: ~5500 t/s (engine 0.1.49.x plateaued at ~3100 — a pure engine-update win).

**Why can decode be *faster* at longer context?** Because this is a streaming MoE: a long prompt's prefill sweeps nearly the entire expert set into page cache, so decode then runs against a hot working set — while short prompts decode against partly-cold experts. Same direction on both quants and across engine versions; it is a real property of expert streaming on this machine, not a measurement artifact.

**Verdict:** UD-IQ3_XXS wins exactly where long context matters (+28% at 262K, +19% at 131K) and is ~15–30% slower for short chat — so IQ3_S stays on the box as the "sprinter" mode.

**Quality:** KL to Q8_0 from Strata's published quant studies (these KL figures are Niko's, *not measured on this box*): IQ3_S 0.144 → UD-IQ4_XS 0.101, **lower = more faithful**. KL of UD-IQ3_XXS itself I did not measure; by quant size it sits between the two. The speed table above is my own measurement.

## The failure boundary — quantified

**UD-IQ4_XS (94 GB of files) does not fit this machine: unusable.** Its expert streams exceed what 63 GB RAM can hold as page cache, so fresh conversations keep re-reading NVMe: 35–81 t/s at short/medium context, 5 t/s on first contact after load. The long-context rows (160 @32K, 211 @262K) look survivable only because the benchmark replays the *same* prompt — a permanently hot expert set; real multi-topic chat does not get that free ride. This quant belongs on ≥96 GB RAM machines.

Other verified dead ends:

- **No `--mmap-experts`**: the engine reads its experts into *anonymous* (unreclaimable) RAM → ~6 GB headroom, swap thrash, multi-minute teardown after stop. With it: page-cache experts, ~40 GB headroom, instant release. When memory behaves oddly, diff `/proc/<pid>/cmdline` against the working config first.
- **DeepSeek-V4.1-Flash (274 GB)** on 63 GB RAM: 9 t/s decode, RAM-bandwidth bound — removed; documented in [the archived repo](https://github.com/RobertDingenouts/local-llm-benchmarks).
- **Unsloth Dynamic files have no native MTP layers** → an MTP route silently runs its weaker suffix drafter (32–42% acceptance); DFlash2 is the only accelerator that engages.
- **`--ssd-cache` is not read-ahead**: it is a RAM store for CPU-lane experts; on this box it *reduced* decode (9.1 → 6.4 t/s) by evicting page cache. (DeepSeek-era finding, kept as a naming lesson.)

## Side-by-side multimedia on one GPU

With `strata lite` (VRAM reserve 10.5 GB) running, measured VRAM deltas while chat stays live:

| Workload | Δ VRAM peak | Result |
|---|---|---|
| TTS (IndexTTS 2.5) | +3.1 GB | live |
| Image gen (Qwen-Image / MOSS turbo) | +10.0 GB | live |
| Music (YuE2) | +9.4 GB | live |

"Not dropped" ≠ "not slower": decode throughput under concurrent tool load is the open follow-up — run `stbench quick` while a tool is up; numbers land here once measured.

## Method

`scripts/stbench` (included): temperature 0, server-side engine clocks, two runs per point by default (raise with `ST_RUNS=5`), medians reported and raw samples kept; RAM guard; refuses to touch servers it did not start. `scripts/udab` chains the full A/B matrix unattended. Raw reports behind every number above: `results/`.

## Credits & licenses

- **[Strata](https://github.com/Niko1221/Strata)** (MIT) by Niko1221 — the inference engine that makes this MoE streaming practical on one GPU.
- **[unsloth/Qwen3.8-Flash-Next-GGUF](https://huggingface.co/unsloth/Qwen3.8-Flash-Next-GGUF)** — Dynamic v3.1 quantizations. Weight license on HF: **"other"** — check upstream terms before commercial use.
- **[incoai/Qwen3.8-Flash-Next-DFlash2](https://huggingface.co/incoai/Qwen3.8-Flash-Next-DFlash2)** (Apache-2.0) — the draft model; the selq4 codebook conversion follows nazeshinjite's community GGUF (Apache-2.0).
- The Qwen team for the base model.

**Old experiments** (llama.cpp/ExLlama/SGLang-era benchmarks) live in [local-llm-benchmarks](https://github.com/RobertDingenouts/local-llm-benchmarks), archived 2026-11 — this repo replaces it. **Stack beyond the model:** Hermes agent gateway (Telegram), Open WebUI, a live HUD dashboard and self-built image/music/voice UIs — this repo focuses on the inference core.

*License: MIT — 2026 Robert Dingenouts (Robas).*
