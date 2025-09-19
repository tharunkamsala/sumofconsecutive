# Gossip & Push-Sum Simulator (Gleam / BEAM)

A production-style Gleam application that implements the classic Gossip and Push-Sum distributed algorithms on top of the BEAM virtual machine. Each simulated node runs as a lightweight Erlang process, exchanges messages with its neighbours, and reports convergence back to a supervising coordinator. The command-line interface exposes multiple topologies, deterministic failure injection, and CSV-friendly metrics so you can explore fault-tolerant information dissemination at scale.

## Table of Contents
- [Key Capabilities](#key-capabilities)
- [Architecture Overview](#architecture-overview)
- [Prerequisites](#prerequisites)
- [Environment Setup](#environment-setup)
- [Build & Test](#build--test)
- [Running Simulations](#running-simulations)
- [Sample Output & Metrics](#sample-output--metrics)
- [Failure Model](#failure-model)
- [Experimentation Workflow](#experimentation-workflow)
- [Configuration Reference](#configuration-reference)
- [Troubleshooting](#troubleshooting)
- [Extending the Simulator](#extending-the-simulator)

## Key Capabilities
- Actor-based implementations of both Gossip and Push-Sum protocols.
- Built-in topologies: `full`, `line`, `3D`, and `imp3D` (3D grid with an additional random neighbour).
- Deterministic pseudo-random generator to guarantee reproducible runs.
- Configurable failure injection via `fail=<rate>` with supervisor-driven convergence targets.
- Metrics emission designed for downstream analysis and automated benchmarking.

## Architecture Overview
- `src/main.gleam` – CLI entry point responsible for topology creation, configuration parsing, and metrics reporting.
- `src/actors.gleam` – Node actor implementation covering gossip dissemination, push-sum averaging, and failure logic.
- `src/coordinator.gleam` – Supervisor actor that tracks live/converged nodes and stops the run when targets are reached.
- `src/topology.gleam` – Graph constructors and degree statistics for each supported layout.
- `src/rand_utils.gleam` – Deterministic RNG helpers shared across nodes and topology builders.
- `test/` – Placeholder gleeunit suite ready to be expanded with deterministic integration tests.

## Prerequisites
- [Gleam](https://gleam.run) ≥ 1.0 (install via Homebrew, asdf, or a standalone binary).
- Erlang/OTP ≥ 26.
- macOS, Linux, or Windows via WSL.
- No additional Hex dependencies beyond those tracked in `manifest.toml`.

## Environment Setup
Verify the toolchain immediately after installation:
```bash
gleam doctor
```

## Build & Test
Run the following from the project root (`project2/`):
```bash
gleam deps download   # Fetch Hex dependencies
gleam build           # Type-check and compile the project
gleam test            # Execute gleeunit tests (currently placeholders)
```

## Running Simulations
The CLI accepts three required positional arguments and an optional failure flag:
```bash
gleam run -- <num_nodes> <topology> <algorithm> [fail=<0.0-1.0>]
```
- `num_nodes` – Positive integer describing how many processes to spawn.
- `topology` – One of `full`, `line`, `3D`, or `imp3D`.
- `algorithm` – `gossip` or `push-sum`.
- `fail` – Optional probability (per scheduling tick) that permanently kills a node. Defaults to `0.0`.

### Reference Scenarios
```bash
# Gossip without failures
gleam run -- 20 line gossip
gleam run -- 50 full gossip
gleam run -- 64 3D gossip
gleam run -- 64 imp3D gossip

# Gossip with failures
gleam run -- 20 line gossip fail=0.2
gleam run -- 50 full gossip fail=0.05
gleam run -- 64 3D gossip fail=0.1
gleam run -- 64 imp3D gossip fail=0.1

# Push-Sum without failures
gleam run -- 20 line push-sum
gleam run -- 50 full push-sum
gleam run -- 64 3D push-sum
gleam run -- 64 imp3D push-sum

# Push-Sum with failures
gleam run -- 20 line push-sum fail=0.1
gleam run -- 50 full push-sum fail=0.05
gleam run -- 64 3D push-sum fail=0.1
gleam run -- 64 imp3D push-sum fail=0.1
```

## Sample Output & Metrics
```text
Topology: line
Nodes: 20
Edges: 19
Degree range: 1 - 2
Average degree: 1.90
Failure rate: 0.20
Converged target: 1/20
Converged nodes: 1/3
Dead nodes: 17
Algorithm: gossip, Topology: line
Elapsed (ms): 409
metrics,20,line,gossip,0.2,completed,409,17,1,1
409
```
- `metrics,...` exposes `num_nodes, topology, algorithm, fail_rate, status, elapsed_ms, dead_nodes, done_nodes, target` for ingestion into dashboards or spreadsheets.
- The trailing integer is maintained for legacy scripts that expect an elapsed time only.
- Runs with high `fail` values (for example `fail=0.2`) converge slowly or not at all because most actors die before steady state.

## Failure Model
- **Trigger** – Before each receive loop, a node samples `rand_utils.next_float` and marks itself dead if the value is `< fail_rate`.
- **Teardown** – Dead nodes emit `NodeDead(id)` exactly once, stop processing their mailbox, and halt gossip/push-sum participation.
- **Supervisor accounting** – For every `NodeDead`, the coordinator recomputes `target = max(1, ceil(0.6 * alive))` where `alive = total_nodes - dead`. If all nodes die, the run terminates with status `timeout`.
- **Converged-but-dead** – Nodes that converged prior to failing are still counted in the final metrics.
- **Deterministic chaos** – Node RNG seeds follow `id * 7919 + 7`, guaranteeing reproducible failure patterns unless the seeding function is changed.
- **Mitigation tips** – Lower `fail`, relax convergence thresholds, or extend the timeout to reduce zero-convergence outcomes with aggressive failure rates.

## Experimentation Workflow
1. Append metrics from batch runs to a CSV:
   ```bash
   for rate in 0.00 0.05 0.10 0.15 0.20; do
     gleam run -- 200 full gossip fail=$rate >> results.csv
   done
   ```
2. Load `results.csv` into your analysis tool of choice (spreadsheet, `gnuplot`, `R`, etc.).
3. Chart convergence time versus `fail_rate`, success rate (`status == completed`), and node mortality to quantify resilience.
4. Summarise methodology and findings in `Report-bonus.pdf` and package the repository as `project2-bonus.tgz` for the bonus submission.

## Configuration Reference
- **Gossip threshold** – Defined in `main.gleam`; defaults to `2`.
- **Push-Sum epsilon** – `1.0e-10` in `actors.gleam`; adjust to trade convergence speed for precision.
- **Timeout** – Global 3 s cap in `wait_for_done_with_timeout`; increase for larger networks.
- **Failure rate** – CLI `fail=<rate>` parameter validated in `[0.0, 1.0]`.
- **RNG seeding** – Update `rand_utils` if non-deterministic runs are desired.

## Troubleshooting
- `gleam` not found – Confirm the executable is on your `PATH`; restart the shell after installation.
- `erlang` missing – Install Erlang/OTP ≥ 26 (macOS: `brew install erlang`, Ubuntu: `sudo apt install erlang`).
- Timeouts without convergence – Try denser topologies (`full`, `imp3D`), increase the timeout, or lower the failure rate.
- Excessive node deaths – Reduce `fail`, decrease the gossip threshold, or relax push-sum epsilon to converge faster.

## Extending the Simulator
- Introduce new topologies by extending `Topology` and registering builders in `src/topology.gleam`.
- Experiment with richer failure modes (temporary link loss, recoveries) and feed additional metrics back to the coordinator.
- Expand the gleeunit suite with deterministic integration scenarios that assert on emitted metrics.
- Surface additional observability outputs (JSON, Prometheus) from `main.gleam` for richer analysis pipelines.

Happy simulating!

