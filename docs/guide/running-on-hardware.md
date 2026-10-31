# Running on hardware

A campaign runs every test program of a corpus until it either sees the test's weak outcome or reaches a
run budget. Use these commands:

```bash
make all
export RESULTS=hetlitmus/run-out/<tag>
sh hetlitmus/probe-cuda.sh
hetlitmus/emit-het.sh --gpu-target cuda _build/default/hetlitmus/tests/het
hetlitmus/build.sh $RESULTS/emit
python3 hetlitmus/campaign.py --corpus $RESULTS/emit --budget-runs 100 --state $RESULTS/campaign.csv
```

Optionally, the stress parameters can be tuned before the campaign, since the values that work
best differ between machines. See [Tuning stress parameters](tuning-stress.md).

For AMD, run `probe-hip.sh` and `emit-het.sh --gpu-target hip`. Use the `het` corpus on an
AArch64 host and `het-x86_64` on an x86_64 host.

## 1. Probe

`probe-cuda.sh` records the GPU, its suggested architecture and the memory attributes that decide
how CPU and GPU share memory. `build.sh` uses that architecture unless you pass `--arch`.

## 2. Emit and build

`emit-het.sh` runs `litmus7` once per test into `$RESULTS/emit`. `build.sh` runs `make clean` and
the link target in every test program directory. Both take `--tests` with a comma-separated list or a
file of names to work on a subset.

## 3. Run a campaign

`campaign.py` invokes each test program repeatedly. One invocation performs 10 runs of 100,000
iterations. A test stops as soon as one clean run shows the weak outcome, or when it reaches
`--budget-runs` runs.

The state CSV is rewritten after every test, one row per test:

| Column | Meaning |
|---|---|
| `test` | test name |
| `stop` | `OBSERVED`: a clean run showed the weak outcome. `BUDGET`: the run budget ran out without it. `ERROR`: the test program failed, timed out or had no usable run |
| `invocations` | how many times `campaign.py` started the test program |
| `seed0` | the seed base of the campaign |
| `runs`, `usable` | the number of runs performed, and of these the runs in which the rendezvous held and every requested stress and noise mechanism ran (usable) |
| `k`, `k_eff` | the number of runs that showed the weak outcome, and of these the runs that were not degenerate (`k_eff`). A run is degenerate when the rendezvous failed, no iteration was scored or every iteration read the same values |
| `scored`, `discarded` | iterations counted, and iterations discarded because the CPU and GPU threads did not meet at the rendezvous within a polling cap |
| `note` | a short explanation of the stop |

## Reading a test program's output

Each invocation ends with a summary line that `campaign.py` parses:

```
HetStats <test> obs=<class> R=<runs> usable=<n> k=<n> k_eff=<n> degen=<n> N=<iterations per run> scored=<n> discarded=<n>
```

`obs` is `Never` when no run showed the weak outcome, `Always` when every run did, and `Sometimes`
otherwise. It is `VOID` when no run was usable. Per-run `HetVerdict` lines above the summary name
the mechanism that made a run unusable.

## Shared memory

The test program allocates the shared locations according to `HET_ALLOC`:

| Value | Allocation |
|---|---|
| `auto` (default) | `malloc` if the GPU can address pageable host memory, else `managed` |
| `malloc` | ordinary host memory; needs a GPU that reports pageable memory access |
| `managed` | CUDA/HIP managed memory |
| `pinned` | pinned host memory |

The test program prints the chosen mode and the device attributes at start-up.

## Reproducing the DGX Spark results

[`runbook.sh`](https://github.com/TUM-DSE/hetlitmus/blob/main/hetlitmus/device-runs/gb10/runbook.sh)
lists the steps of the DGX Spark runs reported in the thesis; the raw data is in
[hetlitmus-gb10-run-data](https://github.com/lenguyenhcm325/hetlitmus-gb10-run-data) repository. For another
CPU-GPU system, keep the order of the steps and adapt the commands in the runbook.
