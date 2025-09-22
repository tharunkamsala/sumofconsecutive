# Gossip & Push-Sum Simulator (Gleam / BEAM)

## Team
- Tharun Kamsala
- Geetha Sravya Somu

## Project Overview
This project simulates large-scale information dissemination using two classic distributed algorithms—**gossip** (a rumour-spreading protocol) and **push-sum** (an averaging protocol). Every simulated node is implemented as its own actor (a lightweight Erlang process) running on the BEAM virtual machine. Nodes exchange messages with their neighbours according to a chosen network topology and report convergence back to a supervising coordinator. The simulator can optionally inject permanent node failures so you can observe how resilience changes with different layouts.

### Key Highlights
- Pure actor-model implementation in Gleam; no shared memory or manual threading.
- Topology builders for `full`, `line`, `3D`, and `imp3D` (3D grid plus a random shortcut) networks.
- Deterministic pseudo-random number generation so experiments are reproducible.
- Command-line interface that emits both human-readable logs and a single machine-friendly CSV metrics line at the end of every run.

## How the Simulator Works
### Actors and Coordinator
- Each node is an Erlang process spawned from `actors.gleam` that owns its state and mailbox.
- A single coordinator actor (in `coordinator.gleam`) tracks how many nodes have converged or died and decides when to stop the run.
- The coordinator requires **60% of the currently alive nodes** to report completion before declaring success.

### Algorithms
- **Gossip**: A node forwards a rumour until it has heard it twice (`gossip_threshold = 2`).
- **Push-Sum**: Nodes continuously exchange `(s, w)` pairs. Convergence is detected when the running average stabilises within `1.0e-10` for three consecutive rounds.

### Failure Model
- Before every mailbox receive, each node rolls a random number; if the value is `< fail_rate`, the node dies permanently and notifies the coordinator exactly once.
- Dead nodes stop sending/receiving protocol messages but still count towards the final statistics.
- When every node has died, the coordinator emits a timeout with zero completed actors.

## Getting Started
### Prerequisites
- [Gleam](https://gleam.run) ≥ 1.0
- Erlang/OTP ≥ 26
- macOS, Linux, or Windows (WSL) shell

### Environment Check
```bash
gleam doctor
```

### Build & Test
From the project root (`project2/`):
```bash
gleam deps download   # Install Hex dependencies
gleam build           # Compile the project
gleam test            # Placeholder gleeunit suite
```

## Running Simulations
Use the executable produced by `gleam run`. The CLI accepts three positional arguments plus an optional failure flag:

```bash
gleam run -- <num_nodes> <topology> <algorithm> [fail=<rate>]
```

Argument | Meaning
-------- | -------
`num_nodes` | Positive integer specifying how many node actors to spawn.
`topology` | One of `full`, `line`, `3D`, `imp3D`.
`algorithm` | Either `gossip` or `push-sum`.
`fail=<rate>` | Optional permanent failure probability per scheduling tick (0.0–1.0). Default is 0.0.

### Example Runs
There are many ways to combine the knobs, so here is a quick command cheat-sheet. All commands assume you are inside the `project2/` directory.

#### Gossip (no failures)
- `gleam run -- 20 line gossip`
- `gleam run -- 50 full gossip`
- `gleam run -- 64 3D gossip`
- `gleam run -- 64 imp3D gossip`

#### Gossip (with failures)
- `gleam run -- 20 line gossip fail=0.05`
- `gleam run -- 50 full gossip fail=0.05`
- `gleam run -- 64 3D gossip fail=0.10`
- `gleam run -- 64 imp3D gossip fail=0.10`
- `gleam run -- 640 imp3D gossip fail=0.10`

#### Push-Sum (no failures)
- `gleam run -- 20 line push-sum`
- `gleam run -- 50 full push-sum`
- `gleam run -- 64 3D push-sum`
- `gleam run -- 64 imp3D push-sum`

#### Push-Sum (with failures)
- `gleam run -- 20 line push-sum fail=0.02`
- `gleam run -- 50 full push-sum fail=0.05`
- `gleam run -- 64 3D push-sum fail=0.05`
- `gleam run -- 64 imp3D push-sum fail=0.05`

Below are two representative outputs.

**1. Gossip on a 10-node line (no failures)**
```bash
gleam run -- 10 line gossip
```
Output excerpt:
```
=== Simulation Setup ===
Algorithm selected: gossip
Network topology: line
Total actors (nodes): 10
Total communication links (edges): 9
Neighbour count range (min-max degree): 1 - 2
Average neighbours per node: 1.8
Failure probability per message exchange: 0.0
Seeding the initial rumor at nodes: 9, 5, 0 (0-based indices)

=== Convergence Summary ===
Actors required to finish: 6 out of 10 total
Actors that actually finished: 6/10
Dead nodes: 0
...
metrics,10,line,gossip,0.0,completed,51,0,6,6
```

**2. Gossip on a 64-node imperfect 3D grid with failures**
```bash
gleam run -- 64 imp3D gossip fail=0.1
```
Output excerpt:
```
=== Convergence Summary ===
All actors crashed before reaching the convergence goal.
Completed actors: 0/0
Dead nodes: 64
...
metrics,64,imp3D,gossip,0.1,timeout,1172,64,0,0
```
### Understanding the Output
Each run prints three sections:
1. **Simulation setup** – Echoes the chosen algorithm/topology, node count, edge statistics, and failure rate. The seed list identifies which nodes start the rumour or push-sum protocol.
2. **Convergence summary** – Reports whether the target was met. If some nodes survive, you’ll see `Actors that actually finished: <done>/<alive>`. If every node dies, a timeout summary is emitted instead.
3. **Metrics lines** – Two final lines designed for scripts:
   - `metrics,<num_nodes>,<topology>,<algorithm>,<fail_rate>,<status>,<elapsed_ms>,<dead>,<done>,<target>`
   - `<elapsed_ms>` again for legacy tooling.

`status` is either `completed` (≥60% of alive nodes converged) or `timeout` (coordinator never reached the target before every node died or the global timeout expired).

## Failure-Rate Experiments

# Failure-Rate Metrics Report

This document summarises the latest batch of simulator runs that sweep both algorithms (`gossip`, `push-sum`), all four topologies, and failure probabilities `0.00`, `0.05`, and `0.10`. Every run spawned 64 actors (one per node) and the metrics were captured from the `metrics,<...>` line that the CLI prints at the end of execution. The raw data lives in `metrics.csv` and the derived plots in `graphs/`.

## Dataset Snapshot
| Algorithm | Topology | Fail Rate | Elapsed (ms) | Dead Nodes | Done Nodes | Target |
| --------- | -------- | --------- | ------------ | ---------- | ---------- | ------ |
| gossip | full  | 0.00 | 204  | 0  | 38 | 38 |
| gossip | full  | 0.05 | 357  | 26 | 22 | 22 |
| gossip | full  | 0.10 | 1172 | 64 | 0  | 0  |
| gossip | line  | 0.00 | 561  | 0  | 38 | 38 |
| gossip | line  | 0.05 | 1835 | 53 | 6  | 6  |
| gossip | line  | 0.10 | 1173 | 64 | 0  | 0  |
| gossip | 3D    | 0.00 | 255  | 0  | 38 | 38 |
| gossip | imp3D | 0.05 | 408  | 21 | 25 | 25 |
| push-sum | full  | 0.05 | 3416 | 64 | 0  | 0  |
| push-sum | full  | 0.10 | 1988 | 64 | 0  | 0  |
| push-sum | line  | 0.05 | 561  | 39 | 15 | 15 |
| push-sum | imp3D | 0.05 | 2249 | 61 | 1  | 1  |

(See `metrics.csv` for the complete 24-row table.)

## Generated Plots
- `graphs/gossip_elapsed.png` – elapsed time versus failure rate, with one series per topology.
- `graphs/push_sum_elapsed.png` – same view for the push-sum algorithm.
- `graphs/topology_full_elapsed.png`, `topology_line_elapsed.png`, `topology_3D_elapsed.png`, `topology_imp3D_elapsed.png` – each compares gossip and push-sum on a fixed topology.

## Observations
- **Dense topologies amplify failure exposure.** For both algorithms the full mesh delivers the fastest spread when `fail=0.00`, but a small failure rate swings the curve dramatically. Push-sum on the full topology jumps from 6 ms at `fail=0.00` to 3.4 s at `fail=0.05` and keeps all nodes alive only because the convergence target collapses to zero when every actor dies. This confirms that the aggressive fan-out causes more “failure rolls” and therefore more casualties.
- **Gossip “completions” can mask total collapse.** At `fail=0.10` every topology reports `status=completed` even though `dead_nodes=64` and `done_nodes=0`. The coordinator recomputes the target as 60% of the surviving nodes; once every node has failed the target becomes zero, so the run terminates as “completed”. When analysing the plots or CSV you should always cross-check `dead_nodes`/`done_nodes` to avoid treating these outcomes as successful spreads.
- Despite the resilience penalty, the imperfect 3D grid (`imp3D`) offers the best middle ground for push-sum: at `fail=0.05` it finishes in ~2.2 s with at least one converged survivor, while every other topology loses the entire population at the same rate.

## Reproducing the Dataset
1. Ensure you are in the project root: `cd project2`.
2. Clear or back up the existing CSV: `rm -f metrics.csv`.
3. Re-run the sweep: `for algo in gossip push-sum; do for topo in full line 3D imp3D; do for fail in 0.0 0.05 0.1; do ...; done; done; done` (see `history` or reuse the loop listed in this file).
4. Plot the refreshed data: `python3 scripts/plot_metrics.py --input metrics.csv`.

The script will regenerate all PNGs under `graphs/`. Feel free to extend the CSV with additional failure rates, node counts, or algorithms and rerun the plotting utility to expand the analysis.

## Repository Layout
Path | Purpose
---- | -------
`src/main.gleam` | CLI entry point, argument parsing, orchestration, and logging.
`src/actors.gleam` | Implementation of gossip and push-sum actors plus failure handling.
`src/coordinator.gleam` | Supervisor actor that stops the run once the convergence target is met.
`src/topology.gleam` | Builders for line, full mesh, 3D grid, and imperfect 3D graph structures.
`src/rand_utils.gleam` | Deterministic pseudo-random helpers and list selection utilities.
`scripts/generate_failure_plots.py` | Batch runner that collects metrics and renders line plots.
`test/` | Placeholder Gleam test suite (ready for future assertions).

## Troubleshooting & Tips
- **`Module 'project2' was not found`** – Ensure `src/project2.gleam` exists; it simply re-exports `main.main()` for Gleam’s build system.
- **`gleam` or `erl` not found** – Add both executables to your `PATH` and rerun `gleam doctor`.
- **Runs never finish** – Dense topologies (`full` or `imp3D`) generally converge faster. Extremely high `fail` values can kill the entire network; try lowering the probability or increasing the timeout in `main.gleam`.
- **Reproducibility** – The simulator seeds each node’s RNG deterministically (`seed = id * 7919 + 7`), so repeated runs with the same parameters produce consistent behaviour.

## Extending the Simulator
- Add topologies by extending the `Topology` type and providing a builder in `topology.gleam`.
- Adjust gossip thresholds, push-sum epsilon, or coordinator target percentage directly in `main.gleam`.
- Implement richer failure modes (transient crashes, recoveries) by editing the failure logic in `actors.gleam`.
- Expand the test suite with deterministic scenarios that assert on the emitted metrics line.

Enjoy experimenting with distributed algorithms on the BEAM!
