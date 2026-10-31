# litmus7

HetLitmus extends `litmus7`, the herdtools7 tool that runs litmus tests on real hardware, with a
back end that compiles a `Het` test into a CUDA or HIP test program.

```
litmus7 -set-libdir litmus/libdir -gpu-target cuda|hip -o DIR test.litmus
```

## litmus7 options

| Option | Meaning |
|---|---|
| `-gpu-target <target>` | the GPU dialect to render: `cuda` or `hip`; no default |
| `-o DIR` | output directory; the test program is written to `DIR/<test>/` |
| `-set-libdir litmus/libdir` | library directory |
| `-s <n>` | iterations per run, default 100,000 |
| `-r <n>` | runs per invocation, default 10 |

All other upstream `litmus7` options apply unchanged.

## Test program Makefile

Every test program directory that `litmus7` writes has a `Makefile`.

### Targets

| Target | Effect |
|---|---|
| `make`, `make cuda`, `make hip` | compile the objects without linking |
| `make cuda-bin`, `make hip-bin` | compile and link the executable `./<test>` |
| `make clean` | remove the objects and the executable |

### Variables

| Variable | Default | Meaning |
|---|---|---|
| `CUDA_ARCH`, `HIP_ARCH` | `sm_90`, `gfx942` | GPU architecture |
| `NVCC`, `HIPCC` | `nvcc`, `hipcc` | GPU compiler |
| `CC` | `gcc` | compiler for the CPU column |
| `HET_CPU_CFLAGS` | `-march=armv8.3-a` for AArch64, empty for x86_64 | flags for the CPU column, used only on a host of the test's CPU ISA |

Build-time `HET_*` settings go into the GPU compiler variable, for example
`make cuda-bin NVCC="nvcc -DHET_LLC_MB=256"`. The settings are listed in
[Runtime settings](runtime-settings.md).
