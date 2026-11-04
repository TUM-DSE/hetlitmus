# Test format

A heterogeneous test uses the herdtools7 litmus format with the architecture `Het`. Each column is
one proc whose header names the device it runs on. For example, this
[message-passing](#message-passing-mp) test has one proc on the GPU and one on the CPU:

```
Het MP-gc-gpu-plain.fsc
"Heterogeneous MP-gc-gpu-plain.fsc: per-proc device assignment gpu,cpu (cpu=AArch64, gpu=LISA)"
{
1:X1=x;
1:X3=y;
}
 P0:gpu             | P1:cpu      ;
 w[relaxed,gpu] x 1 | LDR W0,[X3] ;
 f[sc,gpu]          | LDR W2,[X1] ;
 w[relaxed,gpu] y 1 |             ;
scopes: (sys (gpu (cta P0)))
exists (1:X0=1 /\ 1:X2=0)
```

## Message passing (MP)

MP is the classic two-thread litmus test. One thread writes the data `x` and then the flag `y`. The
other thread reads `y` and then `x`. The weak outcome is that the reader sees the new flag (`y=1`)
but the old data (`x=0`).

In the test above, the GPU proc P0 writes `x` and `y` with a sequentially consistent fence between them and the CPU proc
P1 reads them.

## Components of a test

`Het <name>`
:   The architecture and the test name.

Initial state `{ ... }`
:   Register setup for the CPU column, for example `1:X1=x` puts the address of `x` in register
    `X1` of proc 1.

Column headers
:   The proc and the device it runs on, for example `P0:gpu`.

CPU column
:   Native assembly, as in an ordinary herdtools7 test for that architecture.

GPU column
:   [LISA](#lisa-and-the-bell-model) instructions, each with a memory order and a scope.

`scopes:`
:   The scope tree of the GPU procs. `(sys (gpu (cta P2) (cta P3)))` places P2 and P3 in different
    thread blocks of one GPU.

`exists`
:   The weak outcome of interest. For MP above: proc 1 reads
    the new `y` (`X0=1`) and the old `x` (`X2=0`).

## LISA and the Bell model

The GPU column is written in LISA (Litmus Instruction Set Architecture), the generic architecture of herdtools7. A LISA instruction is a
load `r`, a store `w` or a fence `f`, with annotations in brackets.

A Bell file declares which annotations are valid. HetLitmus uses
[`hetlitmus/bells/gpu.bell`](https://github.com/TUM-DSE/hetlitmus/blob/main/hetlitmus/bells/gpu.bell):

```
enum memory_order = 'relaxed || 'acquire || 'release || 'acqrel || 'sc
enum scopes = 'cta || 'gpu || 'sys

instructions R[{'relaxed,'acquire,'sc}, scopes]
instructions W[{'relaxed,'release,'sc}, scopes]
instructions F[{'acquire,'release,'acqrel,'sc}, scopes]
```

The file defines five memory orders and three scopes named after PTX. For AMD, the HIP back end
maps them to the HIP scopes `workgroup`, `agent` and `system`. The file also fixes which orders each instruction accepts.
The herd7 manual describes the format in full:
[Bell extensions](https://diy.inria.fr/doc/herd.html#sec:bell).

---

Instead of writing a test with C and CUDA or HIP code by hand, you
[generate it](generating-tests.md) from a compact relaxation cycle.
