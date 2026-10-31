# Generating tests

`hetgen7` creates a heterogeneous test from a [relaxation cycle](#cycles), given once per device.

```bash
hetgen7 -set-libdir herd/libdir -bell hetlitmus/bells/gpu.bell \
  -cpu-arch aarch64 -devices cpu,gpu -name MP -stdout \
  -cpu 'PodWW Rfe PodRR Fre' \
  -gpu 'PodWWRelaxedSysRelaxedSys RfeRelaxedSysAcquireSys PodRRAcquireSysAcquireSys FreAcquireSysRelaxedSys'
```

The [Edge naming](#edge-naming) section below explains how the CPU and
GPU edges are named.

See [Test format](test-format.md) for the structure of the generated test and the
[`hetgen7` reference](../reference/hetgen7.md) for all its command-line options.

## Cycles

The command above describes [message passing (MP)](test-format.md#message-passing-mp) by the cycle
`PodWW Rfe PodRR Fre`. A cycle is a list of edges between memory accesses, written in the diy7 vocabulary. The edges used
by the HetLitmus corpora are:

| Edge | Meaning |
|---|---|
| `PodXY` | an access X followed in program order by an access Y to a different location on the same proc. X and Y are `W` (write) or `R` (read), so `PodWR` is a write followed by a read |
| `PosXY` | the same, with both accesses to the same location |
| `Rfe` | a write on one proc, then a read on another proc that returns the value of that write |
| `Fre` | a read on one proc, then a write on another proc that overwrites the value the read returned |
| `Coe` | a write on one proc, then a write to the same location on another proc that comes after it in coherence order |

The herdtools7 manual explains relaxations and cycles in full:
[Introduction to candidate relaxations](https://diy.inria.fr/doc/gen.html#candidate:intro).

## Edge naming

`-cpu` and `-gpu` list the same edges in the same order. Each side extends an edge name with the
ordering of its accesses, in its device's notation:

=== "CPU (AArch64)"

    | Form | Example | Meaning |
    |---|---|---|
    | plain edge | `PodWW` | plain accesses |
    | `PodXY` + `Q`/`L` per access | `PodWWLL`, `PodRRQQ` | acquire load (`Q`, `LDAPR`), release store (`L`, `STLR`) |
    | `DMB.<kind>dXY` | `DMB.SYdWW` | a `DMB SY`, `ST` or `LD` between the two accesses |

=== "CPU (x86_64)"

    | Form | Example | Meaning |
    |---|---|---|
    | plain edge | `PodWR` | plain accesses |
    | `MFencedXY` | `MFencedWR` | an `mfence` between the two accesses |

=== "GPU (Bell)"

    | Form | Example | Meaning |
    |---|---|---|
    | edge + `<Order><Scope>` of its first access, then of its second access | `RfeRelaxedSysAcquireSys` | relaxed system-scope store, acquire system-scope load |
    | `Fence<Order><Scope>dXY` + `<Order><Scope>` of both accesses | `FenceScGpudWWRelaxedGpuRelaxedGpu` | a device-scope SC fence between two relaxed accesses |

    `<Order>` is one of `Relaxed`, `Acquire`, `Release`, `Acqrel`, `Sc` and `<Scope>` one of `Cta`,
    `Gpu`, `Sys`: the memory orders and scopes of the
    [Bell model](test-format.md#lisa-and-the-bell-model), capitalised.
