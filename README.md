# PEAMBP: A Parallel Evolutionary Algorithm for the Maximum Bisection Problem

This repository provides a **precompiled Linux executable** of PEAMBP, the parallel evolutionary algorithm accompanying our manuscript *A Parallel Evolutionary Algorithm for the Maximum Bisection Problem*, submitted to *Swarm and Evolutionary Computation*.

The maximum bisection problem (MBP) seeks a partition of the vertices of a weighted, undirected graph into two subsets with sizes differing by at most one, maximizing the total weight of edges crossing the partition.

> **Code availability:** The repository currently provides the executable `PEAMBP`. The complete C++ source code will be uploaded to this repository after the manuscript is accepted. Benchmark graph files are not included.

## 1. Experimental environment

The experiments reported in the manuscript were conducted using the following environment:

| Component | Configuration |
| --- | --- |
| Operating system | Ubuntu 22.04.5 LTS (64-bit) |
| Processors | 4 × Intel Xeon Platinum 8260 @ 2.40 GHz |
| CPU resources | 96 physical cores / 192 logical threads |
| Memory | 1 TB RAM |
| Compiler | GNU g++ 11.4.0 |
| Language / build options | C++11; `-O3 -std=c++11 -pthread` |

## 2. Graph input

PEAMBP reads weighted, undirected graphs in the following text format:

```text
n m
u1 v1 w1
u2 v2 w2
...
um vm wm
```

- `n`: number of vertices; `m`: number of edges.
- `u`, `v`: endpoints of an edge, indexed **from 1 to n**.
- `w`: integer edge weight.
- Each subsequent line represents one edge; there must be `m` edge records.

For example, this is a small illustrative graph (not a benchmark instance):

```text
4 4
1 2 3
1 3 2
2 4 5
3 4 1
```

**Important:** The executable constructs the input filename from the graph number. It does **not** accept an arbitrary graph path as a command-line argument. The expected working-directory layout is:

```text
PEAMBP_project/
├── PEAMBP
├── README.md
├── big Group/
│   ├── g1/
│   │   └── g1data.txt
│   ├── g25/
│   │   └── g25data.txt
│   └── g58/
│       └── g58data.txt
└── output/
```

For instance `G58`, the program looks for exactly `big Group/g58/g58data.txt` **relative to the directory from which it is launched**. Folder names and capitalization matter on Linux. Create the `output/` directory before execution; PEAMBP does not create it automatically.

## 3. Running the executable

On Linux, open a terminal in the directory containing `PEAMBP` and prepare the input/output folders:

```bash
chmod +x ./PEAMBP
mkdir -p output
mkdir -p "big Group/g58"
# Place the G58 graph file at "big Group/g58/g58data.txt".
```

### Command syntax

```bash
./PEAMBP <graph_number> <runs> <cutoff_seconds> <threads> <task_id> [seed_start]
```

| Argument | Description |
| --- | --- |
| `graph_number` | G-set instance number, e.g. `58` for `G58`. |
| `runs` | Number of consecutive, independent runs within this invocation. |
| `cutoff_seconds` | Wall-clock search time limit **per run**, in seconds. |
| `threads` | Number of PEAMBP worker threads used in each run. |
| `task_id` | User-defined label embedded in output filenames; use a simple label without spaces. |
| `seed_start` | Optional first run identifier; defaults to `1`. Run identifiers are `seed_start`, ..., `seed_start + runs - 1`. |

All numeric arguments must be positive integers. `task_id` is an output label, **not** a random seed.

### Examples

```bash
# Quick functionality check on G1 (illustrative settings, not a paper experiment):
./PEAMBP 1 1 10 2 demo 1

# Main paper configuration on the large instance G58:
# The original experiment runner launched one process per run identifier.
# Run 1 of 16, using 16 threads and a 500 s cutoff:
./PEAMBP 58 1 500 16 PEAMBP16_seed001 1

# Run 16 of 16 (the runner also executed runs 2–15 separately):
./PEAMBP 58 1 500 16 PEAMBP16_seed016 16

# Main paper configuration on the medium instance G25, run 1:
./PEAMBP 25 1 200 16 PEAMBP16_seed001 1

# Alternatively, the executable also supports 16 sequential runs in one process:
./PEAMBP 58 16 500 16 main 1
```

The first example requires `big Group/g1/g1data.txt`; the others require the corresponding `g25` or `g58` files. The paper's main experiment used one run per invocation, repeated for run identifiers 1–16 by the experiment runner. The last example is a valid alternative invocation but differs from that scheduling arrangement. `runs` does not specify the thread count, and the cutoff applies separately to every run.

## 4. Random seed configuration

PEAMBP uses `std::mt19937` and initializes a **separate pseudorandom generator for each worker thread**. For a run identifier `s`, thread count `T`, and zero-based worker index `i` (`0 <= i < T`), the worker's seed is:

```text
worker_seed = T * (s - 1) + i + 1
```

For example, with `T = 16`:

| Run identifier `s` | Worker indices | Worker seeds |
| --- | --- | --- |
| 1 | 0–15 | 1–16 |
| 2 | 0–15 | 17–32 |
| 3 | 0–15 | 33–48 |
| 16 | 0–15 | 241–256 |

In the **main comparative experiments**, every algorithm configuration was evaluated with **16 independent runs, using run identifiers 1–16**. The main PEAMBP configuration uses **16 worker threads** (`PEAMBP-16`). The convergence and thread-count experiments used their specified thread counts and independent run identifiers, typically 1–5. When the thread count changes, the worker seeds also change according to the formula above.

Fixed seeds control pseudorandom initialization and choices, but parallel thread scheduling can still affect solution exchange and the resulting search trajectory. Therefore, identical final values are not guaranteed across machines or repeated parallel executions.

## 5. Benchmark instances and time limits

The paper evaluates **71 weighted G-set instances** (`G1`–`G67`, `G70`, `G72`, `G77`, and `G81`). The time limits are applied to each independent run:

| Instance group | Vertex count | Instances | Number | Cutoff per run |
| --- | --- | --- | ---: | ---: |
| Small | `n <= 1000` | G1–G21, G43–G47, G51–G54 | 30 | 100 s |
| Medium | `1000 < n <= 2000` | G22–G42 | 21 | 200 s |
| Large | `n > 2000` | G48–G50, G55–G67, G70, G72, G77, G81 | 20 | 500 s |

The main comparison uses 16 threads and run identifiers 1–16. Thread-count and convergence experiments use the settings described in the manuscript. The time limit is based on **elapsed (wall-clock) time**, not on accumulated CPU time.
