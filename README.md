# Gossip & Push-Sum Simulator (Gleam / BEAM)

An end-to-end Gleam application that models the classic Gossip and Push-Sum distributed algorithms on top of the BEAM virtual machine. Each simulated node runs as a lightweight Erlang process, exchanges messages with its neighbours, and reports convergence back to a supervisor that orchestrates the entire run. The CLI lets you explore different topologies, inject failures, and export metrics for analysis.

## Features at a Glance
- Actor-based implementation of Gossip and Push-Sum protocols.
- Built-in topologies: `full`, `line`, `3D`, and `imp3D` (3D grid plus one random neighbour per node).
- Deterministic pseudo-random generator for reproducible experiments.
- Failure injection via a `fail=<rate>` flag that permanently kills nodes according to a Bernoulli trial.
- Supervisor adjusts convergence targets as nodes die and emits metrics for downstream plotting.

## Requirements
- [Gleam](https://gleam.run) ≥ 1.0 (`brew install gleam`, `asdf install gleam`, or download a binary release).
- Erlang/OTP ≥ 26 (BEAM runtime required by Gleam projects).
- macOS/Linux or Windows via WSL. No additional Hex dependencies beyond the ones vendored in `manifest.toml`.

Run a quick health check once the toolchain is installed:
```bash
gleam doctor
```

## Repository Layout
- `src/main.gleam` – CLI entry point, topology bootstrapper, and metrics reporter.
- `src/actors.gleam` – Gossip/Push-Sum node processes plus failure logic.
- `src/coordinator.gleam` – Supervisor process that tracks convergence and dead nodes.
- `src/topology.gleam` – Graph builders and degree statistics for all supported layouts.
- `src/rand_utils.gleam` – Deterministic RNG helpers used by nodes and topology builders.
- `test/` – Placeholder gleeunit suite (extend as needed).

## Building and Testing
From the project root (`project2/`):
```bash
gleam deps download   # Fetch Hex dependencies
gleam build           # Type-check and compile the project
gleam test            # Run gleeunit tests (currently placeholder)
```

## Running Simulations
The CLI accepts three required positional arguments and an optional failure flag:
```bash
gleam run -- <num_nodes> <topology> <algorithm> [fail=<0.0-1.0>]
```
- `num_nodes` – positive integer specifying how many processes to spawn.
- `topology` – one of `full`, `line`, `3D`, `imp3D`.
- `algorithm` – `gossip` or `push-sum`.
- `fail` – optional probability (per scheduling tick) that a node crashes permanently. Defaults to `0.0`.

### Example Command Matrix
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

### Sample Output
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
- The `metrics,...` line is CSV-friendly and encodes `num_nodes, topology, algorithm, fail_rate, status, elapsed_ms, dead_nodes, done_nodes, target`.
- The final plain integer is preserved for legacy automation that expects only the elapsed milliseconds.
- High failure rates (for example `fail=0.2`) often lead to zero converged nodes because most actors crash before they receive enough rumours or stabilise their push-sum ratios. See the failure deep-dive below for mitigation tips.

## How the Simulation Works
1. **Topology construction** – `main.gleam` builds the selected graph and prints degree statistics. Nodes are labelled `[0, n-1]` and neighbour lists are derived from the topology module.
2. **Coordinator bootstrap** – A supervisor actor tracks convergence. It receives `NodeDone` and `NodeDead` messages and knows the total node count and target percentage (60%).
3. **Node spawn** – Each node actor receives a deterministic RNG seeded by its ID plus configuration such as gossip threshold, epsilon, and failure rate.
4. **Neighbour handshake** – Nodes exchange subjects so they can message each other directly.
5. **Protocol execution** – The CLI seeds initial gossip rumours into three nodes (first, middle, last) or starts push-sum from node zero. Nodes idle-loop aggressively to keep rumours flowing.
6. **Convergence detection** – Gossip convergence happens when a node hears the rumour `gossip_threshold` times (default 2) and notifies the supervisor. Push-Sum convergence requires three successive ratio deltas below epsilon (`1e-10`).
7. **Completion** – When the coordinator sees `>= 60%` of *alive* nodes converged, it replies to the CLI with a `RunStats` record. The CLI prints stats, metrics, and elapsed time.

## Failure Model Deep-Dive
- **Trigger** – Before each message receive+ticker cycle, a node samples `rand_utils.next_float`. If the value is `< fail_rate`, the node is flagged as dead.
- **One-time teardown** – A dying node emits `NodeDead(id)` to the supervisor and stops processing its mailbox. It no longer forwards gossip or participates in push-sum averaging.
- **Supervisor accounting** – For every `NodeDead`, the coordinator recomputes `target = max(1, ceil(0.6 * alive))`, where `alive = total_nodes - dead`. If all nodes die, the run returns immediately with status `timeout`.
- **Converged-but-dead** – Nodes that converged before dying still count toward `done` in the metrics line, which helps diagnose partially successful runs.
- **Deterministic chaos** – Each node’s RNG seed is derived from its ID (`id * 7919 + 7`), so repeated runs with the same `fail` value reproduce the same failure pattern unless you change the seeding formula.
- **Why you might see `Converged nodes: 0/x`** – With aggressive failure probabilities and the 50 ms polling cadence, many nodes flip to the dead state early. Dead nodes stop forwarding messages, isolating survivors and starving them of updates. Lower `fail`, loosen convergence thresholds, or extend the timeout if you want convergence to succeed more often.

## Collecting Experimental Data
1. Append the `metrics` line to a CSV:
   ```bash
   for rate in 0.00 0.05 0.10 0.15 0.20; do
     gleam run -- 200 full gossip fail=$rate >> results.csv
   done
   ```
2. Load `results.csv` into your plotting tool of choice (spreadsheet, `gnuplot`, `R`, etc.).
3. Plot convergence time vs. `fail_rate`, success rate (status == `completed`), and dead node counts.
4. Summarise methodology, plots, and observations in `Report-bonus.pdf` and archive the repository as `project2-bonus.tgz` if you are submitting the bonus assignment.

## Configuration Knobs
- **Gossip threshold** – Adjusted in `main.gleam` when spawning nodes (defaults to 2).
- **Push-Sum epsilon** – `1.0e-10`; change in `actors.gleam` if you need faster/slower convergence.
- **Timeout** – Global cap of 3 s in `wait_for_done_with_timeout`; increase for large graphs.
- **Failure rate** – CLI option `fail=<rate>`; validated between 0.0 and 1.0 inclusive.
- **RNG seeding** – Change the base seed or per-node seed calculation if you want non-deterministic runs.

## Debugging & Troubleshooting
- `gleam` not found – ensure the binary is on your `PATH`; restart the shell if installed via Homebrew/asdf.
- `erlang` missing – install Erlang/OTP ≥ 26 (macOS: `brew install erlang`, Ubuntu: `sudo apt install erlang`).
- Timeouts without convergence – try denser topologies (`full`, `imp3D`), increase the overall timeout, or lower the failure rate.
- Too many node deaths – reduce `fail`, or adjust gossip/push-sum thresholds to converge faster before the failure wave.

## Extending the Project
- Add new topologies by extending `Topology` and providing a builder in `src/topology.gleam`.
- Implement alternative failure models (e.g. temporary link drops, recoveries) by enriching `actors.gleam` and complementing supervisor statistics.
- Enhance testing with gleeunit by driving smaller networks deterministically and asserting on elapsed times or metrics lines.
- Export additional metrics (CSV/JSON) from `main.gleam` if you need richer analysis or integration with plotting scripts.

Happy simulating!
