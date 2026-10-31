# hetgen7

`hetgen7` extends [`diyone7`](https://diy.inria.fr/doc/gen.html#diyone), the herdtools7 generator
of one test from one cycle. It runs the `diyone7` generator once for the CPU procs and once for the
GPU procs, then merges the results into one `Het` test.

For a CPU-only or GPU-only test, use `diyone7` instead.

```
hetgen7 -set-libdir herd/libdir -bell hetlitmus/bells/gpu.bell [options] -cpu <edges> -gpu <edges>
```

## Options

| Option | Meaning |
|---|---|
| `-cpu <edges>` | the cycle in the CPU edge vocabulary |
| `-gpu <edges>` | the same cycle in the Bell vocabulary, every access annotated `<Order><Scope>` |
| `-devices <d0,d1,...>` | per-proc device assignment, default `cpu,gpu`; at least one proc is `gpu` |
| `-cpu-arch <isa>` | ISA of the `cpu` procs: `aarch64` (default) or `x86_64` |
| `-bell <file>` | the scoped Bell model |
| `-show <what>` | list what the Bell model accepts: `edges`, `annotations` or `fences` |
| `-name <name>` | test name |
| `-stdout` | print the test instead of writing a file |
| `-oneloc` | allow tests whose accesses all go to one location, such as `CoRR` |
| `-com <text>` | the comment line of the test |
| `-set-libdir <path>` | library directory, `herd/libdir` |
| `-help` | print the full option list |

All other `diyone7` options apply unchanged.
