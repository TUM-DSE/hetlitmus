# Compiling tests

`litmus7` turns a `Het` test into one self-contained test program directory.

```bash
mkdir -p out
litmus7 -set-libdir litmus/libdir -gpu-target cuda -o out MP.litmus
```

`-gpu-target` is mandatory: `cuda` for NVIDIA, `hip` for AMD. The output
directory must exist.

See the [`litmus7` reference](../reference/litmus7.md) for all its command-line options.

## Building the executable

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

For a whole corpus, run `hetlitmus/emit-het.sh` to emit a test program for every test, then
`hetlitmus/build.sh` to build them all. See [Running on hardware](running-on-hardware.md).
