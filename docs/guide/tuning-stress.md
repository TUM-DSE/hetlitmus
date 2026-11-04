# Tuning stress parameters

Which stress and noise parameters make weak outcomes appear depends on the machine.
`hetlitmus/tune_stress.py` searches for good values on a small tuning set before a campaign. It
is adapted from the [tuning script](https://github.com/reeselevine/cuda-litmus/blob/d107bb5617ee86fea67a851b73565d62d4c2ecf0/tune.sh) of cuda-litmus.

## 1. Choose a tuning set

The tuning set is a text file with one test name per line. Tuning compares configurations by how many
iterations per second show each test's weak outcome. A test that shows it too often or too rarely may give too
little variation in these rates to distinguish configurations. If possible, pick tests that:

- show their weak outcome neither too often nor too rarely under the default stress
- together cover different shapes, device cuts, scopes and orders

The
[DGX Spark tuning set](https://github.com/TUM-DSE/hetlitmus/blob/main/hetlitmus/device-runs/gb10/tuning-set.md)
is an example, with the rationale for its choice of tests.

## 2. Search

```bash
hetlitmus/emit-het.sh --gpu-target cuda _build/default/hetlitmus/tests/het \
  --tests tuning-set.txt -o $RESULTS/tune-emit
python3 hetlitmus/tune_stress.py search --emit-dir $RESULTS/tune-emit \
  --tests tuning-set.txt --out $RESULTS/tune --configs 50
```

In each round, the search draws a random value for every stress parameter, which together form
one configuration. It then rebuilds the tuning tests with those values, runs each test once and
logs how many weak outcomes per second it showed.

The search rebuilds the tests in `--emit-dir` for every configuration, so emit the tuning tests
into a directory of their own, separate from the campaign's `$RESULTS/emit`. `--configs 50` stops
after 50 scored configurations. Without it, the search runs until you stop it, and `--resume`
continues a stopped search.

## 3. Rank

```bash
python3 hetlitmus/tune_stress.py rank --out $RESULTS/tune
```

For each test, `rank` picks the configuration with the most weak outcomes per second and writes
one `winner-<i>.params` file per winning configuration, holding its parameter values as `-D` flags.

## 4. Build with a winner

```bash
FLAGS=$(grep -v '^#' $RESULTS/tune/winner-<i>.params | tr '\n' ' ')
NVCC="nvcc $FLAGS" hetlitmus/build.sh $RESULTS/emit
```

Then [run the campaign](running-on-hardware.md) as usual. With several winners,
build and run once per winner, each time only on the tests that earlier passes did not reveal, as
the [DGX Spark runbook](https://github.com/TUM-DSE/hetlitmus/blob/main/hetlitmus/device-runs/gb10/runbook.sh)
does.
