# Qwen3.8-Flash-Next 177B MoE — daily driver on one RTX 5090

*One machine, 32 GB VRAM, 63 GB system RAM, a 177B-parameter MoE model at 262K context — running as a real daily assistant, with music, image, video and voice pipelines beside it. Everything in this repo is measured on this exact box, with recipes you can reproduce.*

**What this is:** a complete, measured setup for running [Strata](https://github.com/Niko1221/Strata)-served Qwen3.8-Flash-Next (125B active + 51B PLE) locally. Daily driver: **Unsloth Dynamic v3.1 UD-IQ3_XXS with DFlash2 drafting**. This repo documents the final recipe, the full A/B benchmark matrix behind it, and — just as important — the things that *did not* work on 63 GB of RAM.

**Why it exists:** most "huge model on one GPU" posts cherry-pick one number. Here you get the method (temperature 0, repeated runs, medians), the raw reports, the checksums, and the failure boundary quantified exactly: which quant fits 63 GB and which one doesn't.

---

## Hardware

| Component | Spec |
|---|---|
| GPU | NVIDIA RTX 5090 (32 GB VRAM, sm_120) |
| CPU | Intel Core Ultra 9 285K |
| RAM | 63 GB DDR5 |
| Storage | NVMe Gen4 (~5 GB/s effective) |
| OS | Ubuntu, kernel 7.0, CUDA 13.3 |

## The daily driver — exact recipe

- Strata **v0.2.11**, engine package **0.1.57** (CUDA 13 build)
- Model: `unsloth/Qwen3.8-Flash-Next-GGUF`, revision `d96201194f103c906994f283f504209dc8d2a10c`, quant **UD-IQ3_XXS** (~84 GB, n-gram table shared with UD-IQ4_XS via hardlink)
- Drafter: **DFlash2** (`incoai/Qwen3.8-Flash-Next-DFlash2`, revision `a22a1744e85c17364dcc54a6a420af7135d4c7f6`), packed with `tools/dflash2_pack.py --q4_0` — pack SHA-256 `cda40d62…849f1` (matches Strata's pinned hash bit-for-bit; the `-selq4` selector variant `4a42b982…3ecd` is auto-selected when the engine reads Q4_0 selector codebooks)
- Engine flags (the ones that matter):

```
--max-context 262144
--kv int8 --kv-resident 32768
--mmap-experts            # experts as page cache — THE critical flag, see "failure boundary"
--expert-profile expert-profile.bin --expert-cache auto --prefill auto
--draft dflash2 --draft-model …/q4_0-selq4.pack --draft-fallback none --spec 6
STRATA_HC_Q8=1  STRATA_ROUTE_TAIL_SKIP=0
```

- Vision: `mmproj-BF16.gguf`
- `configs/strata-ud-iq3.example.json` is the full service config (API key redacted).

## Measured performance (2026-11-06, temperature 0, median of 2 runs per point)

Decode t/s by context length:

| context | IQ3_S GSQ-RCO (MTP) | UD-IQ4_XS (DFlash2) | **UD-IQ3_XXS (DFlash2)** |
|---|---|---|---|
| 1K   | **222** | 35 ❌ | 189 |
| 4K   | **236** | 64 ❌ | 198 |
| 8K   | **276** | 81 ❌ | 189 |
| 32K  | **282** | 160 | 251 |
| 131K | 232 | 62 ❌ | **277** |
| 262K | 203 | 211 | **261** |

Draft acceptance: IQ3_S/MTP 71% · UD-IQ3_XXS/DFlash2 76% · UD-IQ4_XS/DFlash2 79%.
Prefill plateau on engine 0.1.57: ~5500 t/s (engine 0.1.49.x plateaued at ~3100 — a pure engine update win).

**Verdict:** UD-IQ3_XXS wins exactly where long context matters (+28% at 262K, +19% at 131K) and is ~15–30% slower for short chat — so IQ3_S stays on the box as the "sprinter" mode. Quality ranking (KL → Q8_0): IQ3_S 0.144 > UD-IQ3_XXS > UD-IQ4_XS 0.101 (per Strata's published quant studies).

## The failure boundary — quantified

**UD-IQ4_XS (94 GB expert set) on 63 GB RAM is unusable: 5–80 t/s.** The expert streams don't fit the page cache, every context length re-thrashes NVMe. This is the honest limit of this box: anything above ~67 GB of experts needs ≥96 GB RAM.

Other verified dead ends:

- **No `--mmap-experts`**: the engine reads ~45 GB of experts into *anonymous* (unreclaimable) RAM → 6 GB headroom, swap thrash, multi-minute teardown after stop. With it: page-cache experts, ~40 GB headroom, instant release. Compare `/proc/<pid>/cmdline` against your working config first when memory behaves oddly.
- **DeepSeek-V4.1-Flash (274 GB)** on 63 GB RAM: 9 t/s decode, RAM-bandwidth bound — removed, documented in [the archived repo](https://github.com/RobertDingenouts/local-llm-benchmarks).
- **Unsloth files have no native MTP layers** → the MTP route silently falls back; DFlash2 is the only accelerator that engages. Strata's drafting module additionally requires the *plain* `q4_0.pack` to exist on disk even if the `selq4` variant is used at runtime.
- **`--ssd-cache` is not read-ahead**: it is a RAM store for CPU-lane experts; on this box it *reduced* decode (9.1 → 6.4 t/s) by evicting page cache. (DeepSeek-era finding, kept as a naming lesson.)

## Side-by-side multimedia on one GPU

With `strata lite` (VRAM reserve 10.5 GB) running, measured VRAM deltas while chat stays live:

| Workload | Δ VRAM peak | Result |
|---|---|---|
| TTS (IndexTTS 2.5) | +3.1 GB | live |
| Image gen (Qwen-Image/MOSS turbo) | +10.0 GB | live |
| Music (YuE2) | +9.4 GB | live |

No GPU server juggling needed — one chat engine + one creative tool coexist on 32 GB.

## Method

`scripts/stbench` (included): temperature 0, server-side engine clocks, two runs per data point, medians reported; RAM guard; refuses to touch servers it did not start. `scripts/udab` chains the full A/B matrix unattended. Raw reports for the table above: `results/`.

## Notes

- Old experiments (llama.cpp/ExLlama/SGLang-era benchmarks) live in [local-llm-benchmarks](https://github.com/RobertDingenouts/local-llm-benchmarks), archived 2026-11 — this repo replaces it.
- Stack beyond the model: Hermes agent gateway (Telegram), Open WebUI, a live HUD dashboard, and self-built image/music/voice UIs — this repo focuses on the inference core.

*License: MIT — 2026 Robert Dingenouts (Robas).*
