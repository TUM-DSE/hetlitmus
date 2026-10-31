# Scripts

All scripts are in [`hetlitmus/`](https://github.com/TUM-DSE/hetlitmus/tree/main/hetlitmus). They write into `$RESULTS`, by default
`hetlitmus/run-out/<date>-<host>`.

## probe-cuda.sh

```
sh hetlitmus/probe-cuda.sh
```

Writes `$RESULTS/probe.txt` as `key=value` lines, including the suggested device architecture
that `build.sh` uses. The script takes no options, only environment variables:

| Variable | Default | Meaning |
|---|---|---|
| `RESULTS` | `hetlitmus/run-out/<date>-<host>` | results directory |
| `NVCC` | `nvcc` | CUDA compiler |

## probe-hip.sh

```
sh hetlitmus/probe-hip.sh
```

The same for AMD GPUs, using the ROCm tools. `HIPCC` (default `hipcc`) takes the place of `NVCC`.

## emit-het.sh

```
emit-het.sh --gpu-target cuda|hip CORPUS [--tests LIST|FILE] [-o EMIT]
```

Emits one test program directory per test of `CORPUS`.

| Option | Default | Meaning |
|---|---|---|
| `--gpu-target <target>` | required | `cuda` or `hip` |
| `CORPUS` | required | directory of tests, for example `_build/default/hetlitmus/tests/het` |
| `--tests <list>` | all | tests to emit, as a comma-separated list or a file with one name per line |
| `-o EMIT` | `$RESULTS/emit` | output directory |

## build.sh

```
build.sh EMIT [--tests LIST|FILE] [--arch A] [-j N]
```

Builds every test program directory of `EMIT`, with `make clean` first. One build uses one
architecture.

| Option | Default | Meaning |
|---|---|---|
| `EMIT` | required | directory with one emitted test program directory per test |
| `--tests <list>` | all | tests to build, as a comma-separated list or a file with one name per line |
| `--arch A` | from `$RESULTS/probe.txt` | GPU architecture, for example `sm_90` or `gfx942` |
| `-j N` | number of cores | builds run in parallel |

## campaign.py

```
campaign.py --corpus EMIT --state CSV [options]
```

| Option | Default | Meaning |
|---|---|---|
| `--corpus DIR` | required | directory with one emitted test program directory per test |
| `--state CSV` | required | state file, rewritten after every test; must not exist yet |
| `--budget-runs N` | 100 | maximum runs per test |
| `--rate` | off | if set, a test keeps running after it shows the weak outcome, until it reaches `--budget-runs` |
| `--seed0 N` | random | base seed for the stress schedules. Each invocation derives its own seed from it, so passing the `seed0` of an earlier campaign replays its schedules |
| `--tests <list>` | all | tests to run, as a comma-separated list or a file with one name per line; every name must have a test program directory in `--corpus` |
| `--timeout S` | 900 | time limit in seconds each time `campaign.py` launches a test's executable `./<test>`. A test that exceeds it is marked `ERROR` |
| `--log-dir DIR` | `<state>-logs/`, where `<state>` is the `--state` path without its extension | per-test transcripts, for example in `campaign-logs/` for `--state campaign.csv` |
| `--dry-run` | off | list the tests it would run, and exit |
