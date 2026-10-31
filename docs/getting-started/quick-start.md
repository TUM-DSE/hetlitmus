# Quick start

The steps below generate one
[message-passing](../guide/test-format.md#message-passing-mp) test from a cycle and run it on an
NVIDIA GPU.

!!! info ""

    This page runs one test by hand to introduce the concepts, so the commands are long. For regular use,
    HetLitmus provides scripts that generate, compile and run whole test families on a device. See
    [Running on hardware](../guide/running-on-hardware.md).

Run the commands from the repository root, after [installation](installation.md).

## 1. Generate the test

```bash
hetgen7 -set-libdir herd/libdir -bell hetlitmus/bells/gpu.bell \
  -cpu-arch aarch64 -devices cpu,gpu -name MP -stdout \
  -cpu 'PodWW Rfe PodRR Fre' \
  -gpu 'PodWWRelaxedSysRelaxedSys RfeRelaxedSysAcquireSys PodRRAcquireSysAcquireSys FreAcquireSysRelaxedSys' \
  > MP.litmus
```

`-cpu` and `-gpu` describe the same cycle twice: once in the CPU vocabulary and once in the
[Bell](../guide/test-format.md#lisa-and-the-bell-model) vocabulary, where every access carries an order and a scope.

`-devices cpu,gpu` places proc 0 on the CPU and proc 1 on the GPU.

```title="MP.litmus"
Het MP
"Heterogeneous MP: per-proc device assignment cpu,gpu (cpu=AArch64, gpu=LISA)"
{
0:X1=x;
0:X3=y;
}
 P0:cpu      | P1:gpu              ;
 MOV W0,#1   | r[acquire,sys] r0 y ;
 STR W0,[X1] | r[acquire,sys] r1 x ;
 MOV W2,#1   |                     ;
 STR W2,[X3] |                     ;
scopes: (sys (gpu (cta P1)))
exists (1:r0=1 /\ 1:r1=0)
```

See [Test format](../guide/test-format.md) for an explanation of each component of this file.

## 2. Emit the test program

```bash
mkdir -p out
litmus7 -set-libdir litmus/libdir -gpu-target cuda -o out MP.litmus
```

`litmus7` writes `out/MP/`:

- `MP.cu`: the GPU thread as a CUDA kernel and the host driver that starts both sides and counts
  the outcomes
- `MP_cpu.c`: the CPU thread, as inline assembly
- `het_*.h`: the runtime headers, which synchronise CPU and GPU every iteration, apply stress and
  classify each run
- `outs.c`, `outs.h`: the outcome histogram from litmus7
- `Makefile`, `comp.sh`: build the executable `./MP`

## 3. Build and run

```bash
cd out/MP
make cuda-bin CUDA_ARCH=sm_90
./MP
```

Set `CUDA_ARCH` to your GPU's architecture; the default is `sm_90`.

!!! info ""

    The CPU column is native assembly, so an AArch64 test links only on an AArch64 host such as
    Grace or the DGX Spark. On an x86_64 host, replace `-cpu-arch aarch64` with `-cpu-arch x86_64` in step 1.

By default, one invocation performs 10 runs of 100,000 iterations each and prints the outcome
histogram and summary statistics.
