# stbench 20261106-140230 — Strata iq3s (engine-klok, temp 0, 2x herhaald)

## Decode (128 tok output, temp 0) — per lengte 2x

- **decode-1024#1**: prefill 286.7 · decode 126.9 · in 940 · TTFT 92ms · MTP 93/119
- **decode-1024#2**: prefill 684.4 · decode 222.5 · in 940 · TTFT 25ms · MTP 94/111
- **decode-4096#1**: prefill 668.7 · decode 235.9 · in 3706 · TTFT 26ms · MTP 93/115
- **decode-4096#2**: prefill 1439.5 · decode 196.6 · in 3706 · TTFT 26ms · MTP 86/140
- **decode-8192#1**: prefill 2650.1 · decode 247.5 · in 7393 · TTFT 28ms · MTP 85/117
- **decode-8192#2**: prefill 2764.0 · decode 276.3 · in 7393 · TTFT 28ms · MTP 90/119
- **decode-32768#1**: prefill 5689.6 · decode 232.3 · in 29682 · TTFT 36ms · MTP 68/105
- **decode-32768#2**: prefill 5482.5 · decode 281.9 · in 29682 · TTFT 36ms · MTP 97/121
- **decode-131072#1**: prefill 5916.5 · decode 226.9 · in 119272 · TTFT 67ms · MTP 84/137
- **decode-131072#2**: prefill 5442.4 · decode 231.8 · in 86504 · TTFT 68ms · MTP 81/127
- **decode-262144#1**: prefill 5519.4 · decode 195.6 · in 206636 · TTFT 112ms · MTP 80/123
- **decode-262144#2**: prefill 5435.8 · decode 203.4 · in 206636 · TTFT 107ms · MTP 81/120
## Prefill (1 tok output) — per lengte + 64K

- **prefill-1024**: prefill 379.1 · decode 37.6 · in 924 · TTFT 25ms · MTP 0/0
- **prefill-4096**: prefill 1308.2 · decode 151.5 · in 3690 · TTFT 26ms · MTP 0/0
- **prefill-8192**: prefill 2966.7 · decode 151.5 · in 7377 · TTFT 28ms · MTP 0/0
- **prefill-32768**: prefill 3841.9 · decode 149.3 · in 29666 · TTFT 35ms · MTP 0/0
- **prefill-131072**: prefill 4642.4 · decode 73.0 · in 86488 · TTFT 67ms · MTP 0/0
- **prefill-262144**: prefill 5668.4 · decode 142.9 · in 206620 · TTFT 112ms · MTP 0/0
- **prefill-65536**: prefill 5242.4 · decode 147.1 · in 59362 · TTFT 46ms · MTP 0/0

## Mediaan (van 2 runs)
```
  1024 tok in | decode mediaan  222.5 t/s | prefill   379.1 t/s
  4096 tok in | decode mediaan  235.9 t/s | prefill  1308.2 t/s
  8192 tok in | decode mediaan  276.3 t/s | prefill  2966.7 t/s
 32768 tok in | decode mediaan  281.9 t/s | prefill  3841.9 t/s
131072 tok in | decode mediaan  231.8 t/s | prefill  4642.4 t/s
262144 tok in | decode mediaan  203.4 t/s | prefill  5668.4 t/s
```

Fri Nov  6 02:07:06 PM CET 2026
