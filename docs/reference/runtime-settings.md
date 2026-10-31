# Runtime settings

## Environment variables

Read by the test program at start-up; a change needs no rebuild.

| Variable | Default | Meaning |
|---|---|---|
| `HET_ALLOC` | `auto` | shared-memory allocation: `auto`, `malloc`, `managed` or `pinned`, see [Shared memory](../guide/running-on-hardware.md#shared-memory) |
| `HET_RUNS_MAX` | 10 | runs in this invocation, at most the compiled 10 |
| `HET_STOP_AT_SIGHTING` | 0 | 1 = stop after the first run that shows the weak outcome |
| `HET_SEED` | drawn at random | seed of the stress schedule. The same seed repeats the same schedule |
| `HET_CAP_CPU` | 524288 | polls of the rendezvous counter by a CPU thread before an iteration is discarded |
| `HET_CAP_GPU` | 16384 | polls of the rendezvous counter by a GPU thread before an iteration is discarded |

## Build-time settings

Passed as `-D` in the compiler variable, for example
`make cuda-bin NVCC="nvcc -DHET_LLC_MB=256"`.

| Setting | Default | Meaning |
|---|---|---|
| `HET_LLC_MB` | 114 | last-level cache size in MB; the default is the Grace L3 |
| `HET_NOISE_MB` | 8192 | noise buffer size in MB; must exceed `HET_LLC_MB` |
| `HET_CPU_STRESS_THREADS` | -1 | CPU stress threads; -1 = every spare core |
| `HET_GPU_STRESS_BLOCKS` | -1 | GPU stress blocks; -1 = fill the co-resident grid |
| `HET_CPU_FIRST_CORE` | 0 | core index (from 0) of the first CPU test thread. The other test threads, then the noise threads, then the stress threads take the next indices, one core each |
| `HET_CPU_RESERVE_CORES` | 2 | cores kept free of stress threads, for example for the operating system |
| `HET_CPU_STRESS_PATTERN` | 0 | 0 = st;st, 1 = st;ld, 2 = ld;st, 3 = ld;ld |
| `HET_GPU_MEM_STRESS_PATTERN` | 0 | same encoding, GPU memory stress |
| `HET_GPU_STRESS_ASSIGN` | 1 | 0 = round-robin, 1 = chunking |
| `HET_CPU_PRELOAD_PCT` | 50 | % of iterations with CPU preload |
| `HET_GPU_PRE_STRESS_PCT` | 65 | % of iterations with GPU pre-stress |
| `HET_CPU_NOISE_THREADS` | 0 | CPU noise threads |
| `HET_GPU_NOISE_BLOCKS` | 0 | GPU noise blocks |
| `HET_RDV_MAX_DISCARD_PCT` | 50 | a run that discards more than this % of iterations is unusable |
