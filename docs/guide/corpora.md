# Test corpora

HetLitmus provides three corpora of generated tests:

| Corpus | Tests | Enumeration |
|---|---|---|
| CPU-GPU with AArch64 CPU side | 6,923 | shape × device cut × GPU scope × GPU order × CPU order |
| CPU-GPU with x86_64 CPU side | 4,056 | the same enumeration |
| GPU only | 771 | shape × GPU scope × GPU order |

Unlike the CPU side, the GPU side needs no separate corpus per vendor: it is written in
[LISA](test-format.md#lisa-and-the-bell-model), and `litmus7` then compiles the same test to CUDA or HIP.

The corpus generator script
[`hetlitmus/tests/grid.py`](https://github.com/TUM-DSE/hetlitmus/blob/main/hetlitmus/tests/grid.py)
enumerates them, and `make hetlitmus-corpus-gen` writes them to
`_build/default/hetlitmus/tests/`.

## What is enumerated

Shapes
:   29 cycles: MP, SB, LB, 2+2W, R, S, WRC, RWC, ISA2, IRIW, IRWIW, IRRWIW, 3.2W, WWC, WRW+2W,
    WRW+WR, WRR+2W, W+RWC, Z6.0 to Z6.5, 3.SB, 3.LB, CoRR, CoWR, CoRW2.

Device cuts
:   Every assignment of the procs to CPU and GPU with at least one proc on each device.

GPU scopes
:   `cta`, `gpu`, `sys`.

GPU orders
:   `rlx`, `acq`, `rel`, `ra`, `sc` on the accesses, or a fence between program-ordered accesses:
    `fsc`, `facq`, `frel`, `fra`.

CPU orders
:   AArch64: `plain`, `ra` (`LDAPR`/`STLR`), `sy`, `st`, `ld` (`DMB` variants). x86_64: `plain`,
    `mf` (`mfence`). Orders other than `plain` are combined with
    `sys` scope only, because `sys` is the only scope that includes the CPU.

## Duplicates

Two kinds of duplicates are removed:

- **Symmetric device cuts:** two device cuts that a rotation of the cycle maps onto each other are
  generated only once. For example, the two threads of SB do the same thing, so the device cuts
  CPU-GPU and GPU-CPU give the same test and only `SB-cg-…` is generated.
- **Identical tests:** after generation, a test whose body is identical to an earlier test's is
  dropped. For example, in `MP-cg` the GPU thread only reads, so `rel` (release on stores) gives the
  same test as `rlx` and `MP-cg-sys-plain.rel` is dropped.

## Test names

A test name has five parts, for example `MP-gc-sys-plain.rel`:

| Part | Example | Meaning |
|---|---|---|
| shape | `MP` | the cycle |
| device cut | `gc` | one letter per proc, `c` = CPU, `g` = GPU |
| GPU scope | `sys` | the scope of the GPU accesses |
| CPU order | `plain` | the ordering on the CPU side |
| GPU order | `rel` | the ordering on the GPU side |

Tests of the x86_64 corpus end in `-x86_64`, for example `SB-cg-sys-mf.sc-x86_64`.

GPU-only tests are named `<shape>-<scope>-<order>`, for example `MP-sys-ra`.

## Adding a shape

It is recommended to add a `(name, cycle)` pair to `SHAPES` in
[`hetlitmus/tests/grid.py`](https://github.com/TUM-DSE/hetlitmus/blob/main/hetlitmus/tests/grid.py) and regenerate, rather than calling `hetgen7` by hand.
The generator then produces the new shape across all axes, with naming and deduplication.
