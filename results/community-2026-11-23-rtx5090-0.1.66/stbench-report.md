# stbench 20261123-113416 — Strata udq3 (engine-klok, temp 0, 2x herhaald)

## Decode (128 tok output, temp 0) — per lengte 2x

- **decode-1024#1**: prefill 259.8 · decode 45.3 · in 941 · TTFT 3687ms · MTP 91/125
- **decode-1024#2**: prefill 326.5 · decode 51.5 · in 941 · TTFT 2988ms · MTP 96/132
- **decode-4096#1**: prefill 657.1 · decode 114.1 · in 3707 · TTFT 5691ms · MTP 98/126
- **decode-4096#2**: prefill 627.4 · decode 284.8 · in 3707 · TTFT 5954ms · MTP 97/131
- **decode-8192#1**: prefill 1778.5 · decode 304.4 · in 7394 · TTFT 4201ms · MTP 99/130
- **decode-8192#2**: prefill 1811.3 · decode 342.4 · in 7394 · TTFT 4124ms · MTP 99/133
- **decode-32768#1**: prefill 2527.4 · decode 209.3 · in 29683 · TTFT 11843ms · MTP 100/139
- **decode-32768#2**: prefill 2652.6 · decode 310.9 · in 29683 · TTFT 11312ms · MTP 99/132
- **decode-131072#1**: prefill 3553.4 · decode 120.9 · in 89456 · TTFT 25281ms · MTP 62/91
- **decode-131072#2**: prefill 4038.5 · decode 235.5 · in 89456 · TTFT 22262ms · MTP 62/88
- **decode-262144#1**: prefill 4558.7 · decode 93.9 · in 209588 · TTFT 46194ms · MTP 84/115
- **decode-262144#2**: prefill 5159.6 · decode 244.6 · in 209588 · TTFT 40807ms · MTP 95/130
## Prefill (1 tok output) — per lengte + 64K

- **prefill-1024**: prefill 289.9 · decode 87.7 · in 924 · TTFT 3224ms · MTP 0/0
- **prefill-4096**: prefill n-a-cache(3683) · decode 84.7 · in 7 · TTFT 329ms · MTP 0/0
- **prefill-8192**: prefill n-a-cache(7370) · decode 58.5 · in 7 · TTFT 502ms · MTP 0/0
- **prefill-32768**: prefill n-a-cache(29659) · decode 149.3 · in 7 · TTFT 546ms · MTP 0/0
- **prefill-131072**: prefill n-a-cache(119249) · decode 57.5 · in 7 · TTFT 1289ms · MTP 0/0
- **prefill-262144**: prefill 6149.4 · decode 142.9 · in 209571 · TTFT 34217ms · MTP 0/0
- **prefill-65536**: prefill n-a-cache(59355) · decode 83.3 · in 7 · TTFT 1559ms · MTP 0/0

## Mediaan (van 2 runs; min–max tussen [])
```
  1024 tok in | decode mediaan   51.5 t/s [45-52] | prefill   289.9 t/s
  4096 tok in | decode mediaan  284.8 t/s [114-285]
  8192 tok in | decode mediaan  342.4 t/s [304-342]
 32768 tok in | decode mediaan  310.9 t/s [209-311]
131072 tok in | decode mediaan  235.5 t/s [121-236]
262144 tok in | decode mediaan  244.6 t/s [94-245] | prefill  6149.4 t/s
```


Mon Nov 23 11:38:41 AM CET 2026
