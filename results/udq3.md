# stbench 20261106-141842 — Strata udq3 (engine-klok, temp 0, 2x herhaald)

## Decode (128 tok output, temp 0) — per lengte 2x

- **decode-1024#1**: prefill 300.9 · decode 103.2 · in 940 · TTFT 36ms · MTP 98/124
- **decode-1024#2**: prefill 528.7 · decode 188.7 · in 940 · TTFT 25ms · MTP 97/123
- **decode-4096#1**: prefill 881.6 · decode 96.5 · in 3706 · TTFT 27ms · MTP 90/121
- **decode-4096#2**: prefill 1219.0 · decode 197.7 · in 3706 · TTFT 26ms · MTP 90/131
- **decode-8192#1**: prefill 2490.1 · decode 147.9 · in 7393 · TTFT 27ms · MTP 84/119
- **decode-8192#2**: prefill 2551.6 · decode 188.5 · in 7393 · TTFT 28ms · MTP 98/116
- **decode-32768#1**: prefill 5024.7 · decode 178.7 · in 29682 · TTFT 36ms · MTP 70/95
- **decode-32768#2**: prefill 4272.8 · decode 250.8 · in 29682 · TTFT 35ms · MTP 98/118
- **decode-131072#1**: prefill 4357.7 · decode 133.7 · in 119272 · TTFT 69ms · MTP 63/88
- **decode-131072#2**: prefill 4949.0 · decode 276.5 · in 86504 · TTFT 69ms · MTP 98/118
- **decode-262144#1**: prefill 5391.2 · decode 168.8 · in 206636 · TTFT 107ms · MTP 82/111
- **decode-262144#2**: prefill 5366.4 · decode 261.1 · in 206636 · TTFT 110ms · MTP 96/128
## Prefill (1 tok output) — per lengte + 64K

- **prefill-1024**: prefill 347.6 · decode 156.2 · in 924 · TTFT 26ms · MTP 0/0
- **prefill-4096**: prefill 1505.8 · decode 149.3 · in 3690 · TTFT 26ms · MTP 0/0
- **prefill-8192**: prefill 2794.8 · decode 47.6 · in 7377 · TTFT 28ms · MTP 0/0
- **prefill-32768**: prefill 5320.2 · decode 149.3 · in 29666 · TTFT 37ms · MTP 0/0
- **prefill-131072**: prefill 5564.9 · decode 144.9 · in 86488 · TTFT 69ms · MTP 0/0
- **prefill-262144**: prefill 5547.6 · decode 140.8 · in 206620 · TTFT 106ms · MTP 0/0
- **prefill-65536**: prefill 5158.1 · decode 66.2 · in 59362 · TTFT 45ms · MTP 0/0

## Mediaan (van 2 runs)
```
  1024 tok in | decode mediaan  188.7 t/s | prefill   347.6 t/s
  4096 tok in | decode mediaan  197.7 t/s | prefill  1505.8 t/s
  8192 tok in | decode mediaan  188.5 t/s | prefill  2794.8 t/s
 32768 tok in | decode mediaan  250.8 t/s | prefill  5320.2 t/s
131072 tok in | decode mediaan  276.5 t/s | prefill  5564.9 t/s
262144 tok in | decode mediaan  261.1 t/s | prefill  5547.6 t/s
```

Fri Nov  6 02:23:33 PM CET 2026
