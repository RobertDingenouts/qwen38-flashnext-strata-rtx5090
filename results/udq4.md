# stbench 20261106-140709 — Strata udq4 (engine-klok, temp 0, 2x herhaald)

## Decode (128 tok output, temp 0) — per lengte 2x

- **decode-1024#1**: prefill 74.8 · decode 5.3 · in 940 · TTFT 91ms · MTP 88/113
- **decode-1024#2**: prefill 94.6 · decode 35.1 · in 940 · TTFT 25ms · MTP 90/107
- **decode-4096#1**: prefill 399.3 · decode 59.0 · in 3706 · TTFT 28ms · MTP 100/112
- **decode-4096#2**: prefill 420.7 · decode 63.8 · in 3706 · TTFT 33ms · MTP 90/126
- **decode-8192#1**: prefill 764.1 · decode 81.2 · in 7393 · TTFT 28ms · MTP 100/115
- **decode-8192#2**: prefill 894.1 · decode 59.3 · in 7393 · TTFT 26ms · MTP 90/120
- **decode-32768#1**: prefill 1002.7 · decode 87.8 · in 29682 · TTFT 35ms · MTP 96/120
- **decode-32768#2**: prefill 1040.2 · decode 160.3 · in 29682 · TTFT 35ms · MTP 94/119
- **decode-131072#1**: prefill 1679.0 · decode 53.1 · in 119272 · TTFT 68ms · MTP 94/121
- **decode-131072#2**: prefill 1656.5 · decode 62.4 · in 89064 · TTFT 68ms · MTP 93/124
- **decode-262144#1**: prefill 3338.3 · decode 159.0 · in 209196 · TTFT 108ms · MTP 70/92
- **decode-262144#2**: prefill 3086.8 · decode 211.1 · in 209196 · TTFT 131ms · MTP 63/81
## Prefill (1 tok output) — per lengte + 64K

- **prefill-1024**: prefill 202.5 · decode 33.3 · in 924 · TTFT 39ms · MTP 0/0
- **prefill-4096**: prefill 784.7 · decode 144.9 · in 3690 · TTFT 37ms · MTP 0/0
- **prefill-8192**: prefill 1569.6 · decode 144.9 · in 7377 · TTFT 39ms · MTP 0/0
- **prefill-32768**: prefill 1240.4 · decode 144.9 · in 29666 · TTFT 56ms · MTP 0/0
- **prefill-131072**: prefill 3245.0 · decode 27.5 · in 89048 · TTFT 81ms · MTP 0/0
- **prefill-262144**: prefill 3624.6 · decode 138.9 · in 209180 · TTFT 138ms · MTP 0/0
- **prefill-65536**: prefill 3238.6 · decode 144.9 · in 59362 · TTFT 47ms · MTP 0/0

## Mediaan (van 2 runs)
```
  1024 tok in | decode mediaan   35.1 t/s | prefill   202.5 t/s
  4096 tok in | decode mediaan   63.8 t/s | prefill   784.7 t/s
  8192 tok in | decode mediaan   81.2 t/s | prefill  1569.6 t/s
 32768 tok in | decode mediaan  160.3 t/s | prefill  1240.4 t/s
131072 tok in | decode mediaan   62.4 t/s | prefill  3245.0 t/s
262144 tok in | decode mediaan  211.1 t/s | prefill  3624.6 t/s
```

Fri Nov  6 02:18:38 PM CET 2026
