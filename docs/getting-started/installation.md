# Installation

## Requirements

- OCaml and dune, as in [herdtools7 installation guide](https://github.com/herd/herdtools7/blob/master/INSTALL.md)
- `python3` and `gcc`
- CUDA 12.x (`nvcc`) for NVIDIA GPUs (sm_70 or later) or ROCm 7.2.x (`hipcc`) for AMD GPUs
- A CPU-GPU system with shared memory

## Build

```bash
git clone https://github.com/TUM-DSE/hetlitmus.git
cd hetlitmus
make all
```

`make all` builds every binary into `_build/install/default/bin`.

!!! warning "Use the HetLitmus build"

    A `litmus7` already on your `PATH` may come from another herdtools7 installation without the
    heterogeneous back end. Therefore put the HetLitmus build first on your `PATH`:

    ```bash
    export PATH=$PWD/_build/install/default/bin:$PATH
    ```

## Check the build

```bash
hetgen7 -set-libdir herd/libdir -bell hetlitmus/bells/gpu.bell -show annotations
```

This lists the GPU annotations the [Bell model](../guide/test-format.md#lisa-and-the-bell-model) accepts. Next: [Quick start](quick-start.md).
