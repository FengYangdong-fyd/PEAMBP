# PEAMBP: A Parallel Evolutionary Algorithm for the Maximum Bisection Problem

This repository provides the precompiled Linux executable `PEAMBP` and all 71 G-set benchmark instances used in our paper *A Parallel Evolutionary Algorithm for the Maximum Bisection Problem*, submitted to *Swarm and Evolutionary Computation*. The complete C++ source code will be uploaded to this repository after the manuscript is accepted.

## Environment

Experiments were conducted on Ubuntu 22.04.5 LTS (64-bit), using Intel Xeon Platinum 8260 processors (96 physical cores / 192 logical threads), 1 TB RAM, and g++ 11.4.0.

## How to run

On Linux, open a terminal in the repository's root directory and prepare the executable and output directory:

```bash
chmod +x ./PEAMBP
mkdir -p output
```

Run the program using:

```bash
./PEAMBP <graph_number> <runs> <cutoff_seconds> <threads> <task_id> [seed_start]
```

| Argument | Description |
| --- | --- |
| `graph_number` | G-set instance number (e.g., `58` for G58). |
| `runs` | Number of independent runs in this invocation. |
| `cutoff_seconds` | Time limit in seconds per run (wall-clock time). |
| `threads` | Number of worker threads per run. |
| `task_id` | Label used to distinguish result files; avoid spaces. |
| `seed_start` | Optional starting run identifier (default: `1`). |

The 71 graph files are already included under `big Group/gN/gNdata.txt`, where `N` is the graph number. **Keep this directory structure unchanged** and run the program from the repository root.

For the main paper experiments, use **16 threads** and **16 independent runs**, with `runs=1` and `seed_start` set to **1 through 16**, one invocation for each run. Use a distinct `task_id` for each invocation.

The per-run time limits used in the paper are **100 s** for small instances (`n <= 1000`), **200 s** for medium instances (`1000 < n <= 2000`), and **500 s** for large instances (`n > 2000`).
