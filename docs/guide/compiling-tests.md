# Compiling tests

## Compiling a whole corpus

First generate the corpora with `make hetlitmus-corpus-gen`, as described in
[Test corpora](corpora.md). Then run:

```bash
export RESULTS=hetlitmus/run-out/<tag>
sh hetlitmus/probe-cuda.sh
hetlitmus/emit-het.sh --gpu-target cuda _build/default/hetlitmus/tests/het
hetlitmus/build.sh $RESULTS/emit
```

The command above uses the AArch64 corpus. On an x86_64 host, pass
`_build/default/hetlitmus/tests/het-x86_64` to `emit-het.sh` instead.

For AMD, run `probe-hip.sh` and `emit-het.sh --gpu-target hip`.

`probe-cuda.sh` records the GPU, its suggested architecture (for example `sm_90`) and the memory
attributes that decide how CPU and GPU share memory. `build.sh` uses that architecture unless you pass `--arch`.

`emit-het.sh` runs `litmus7` once per test into `$RESULTS/emit`. `build.sh` runs `make clean` and
the link target in every test program directory. Both take `--tests` with a comma-separated list or a
file of names to work on a subset.

## Compiling a single test

To compile one test by hand, for example while developing a new shape, call `litmus7` directly. It
turns a `Het` test into one self-contained test program directory.

```bash
mkdir -p out
litmus7 -set-libdir litmus/libdir -gpu-target cuda -o out MP.litmus
```

`-gpu-target` is mandatory: `cuda` for NVIDIA, `hip` for AMD. The output
directory must exist.

See the [`litmus7` reference](../reference/litmus7.md) for all its command-line options.

### Building the executable

=== "CUDA"

    ```bash
    cd out/MP
    make cuda-bin CUDA_ARCH=sm_90
    ```

=== "HIP"

    ```bash
    cd out/MP
    make hip-bin HIP_ARCH=gfx942
    ```

The Makefile's targets and variables, including build-time settings, are listed in the
[litmus7 reference](../reference/litmus7.md#test-program-makefile).

One-sided GPU fences (`f[acquire,...]`, `f[release,...]`) need sm_90 or later in CUDA.
