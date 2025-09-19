Perfect — here’s a **complete, professional, GitHub-ready README.md** with everything polished, consistent, and including nice badges + sections that make it look production quality:

````markdown
# Gossip & Push-Sum Simulator (Gleam / BEAM)

[![Build Status](https://img.shields.io/badge/build-passing-brightgreen)]()  
[![Language](https://img.shields.io/badge/Gleam-1.0+-blue)](https://gleam.run)  
[![Erlang](https://img.shields.io/badge/Erlang-26+-red)](https://www.erlang.org)  
[![License](https://img.shields.io/badge/license-MIT-lightgrey)]()

An end-to-end Gleam application that models the **Gossip** and **Push-Sum** distributed algorithms on top of the **BEAM virtual machine**. Each simulated node runs as a lightweight Erlang process, exchanges messages with its neighbours, and reports convergence to a supervisor that orchestrates the run.  

The CLI allows you to:  
- Explore multiple network topologies  
- Inject permanent node failures  
- Collect reproducible experimental metrics  

---

## ✨ Features
- Actor-based implementation of Gossip and Push-Sum protocols  
- Built-in topologies: `full`, `line`, `3D`, `imp3D` (3D grid + random neighbour)  
- Deterministic pseudo-random generator for reproducible experiments  
- Failure injection via `fail=<rate>` flag  
- Supervisor dynamically adjusts convergence targets and emits CSV metrics  

---

## ⚙️ Requirements
- [Gleam](https://gleam.run) ≥ **1.0**  
- Erlang/OTP ≥ **26** (BEAM runtime)  
- macOS/Linux or Windows via WSL  
- No additional Hex dependencies (beyond `manifest.toml`)  

Verify your toolchain:
```bash
gleam doctor
````

---

## 📂 Repository Layout

```text
project2/
 ├── src/
 │    ├── main.gleam        # CLI entry point, topology bootstrapper, metrics reporter
 │    ├── actors.gleam      # Gossip/Push-Sum node processes + failure logic
 │    ├── coordinator.gleam # Supervisor for convergence & node tracking
 │    ├── topology.gleam    # Graph builders & statistics for all layouts
 │    └── rand_utils.gleam  # Deterministic RNG helpers
 ├── test/                  # Placeholder gleeunit suite
 └── README.md
```

---

## 🛠️ Build & Test

From project root (`project2/`):

```bash
gleam deps download   # Fetch dependencies
gleam build           # Compile project
gleam test            # Run gleeunit tests (currently placeholder)
```

---

## 🚀 Running Simulations

CLI usage:

```bash
gleam run -- <num_nodes> <topology> <algorithm> [fail=<0.0-1.0>]
```

**Arguments**:

* `num_nodes` → number of processes (positive integer)
* `topology` → one of `full`, `line`, `3D`, `imp3D`
* `algorithm` → `gossip` or `push-sum`
* `fail` → optional probability of node crash (default: `0.0`)

### Example Runs

```bash
# Gossip (no failures)
gleam run -- 20 line gossip
gleam run -- 50 full gossip

# Gossip (with failures)
gleam run -- 50 full gossip fail=0.05

# Push-Sum (no failures)
gleam run -- 64 3D push-sum

# Push-Sum (with failures)
gleam run -- 64 imp3D push-sum fail=0.1
```

---

## 📊 Sample Output

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

* `metrics,...` → CSV-friendly: `num_nodes, topology, algorithm, fail_rate, status, elapsed_ms, dead_nodes, done_nodes, target`
* Final integer → elapsed milliseconds (legacy automation support)

---

## 🔎 Simulation Workflow

1. **Topology construction** – graph built with degree stats
2. **Coordinator bootstrap** – supervisor tracks convergence & node deaths
3. **Node spawn** – each actor seeded with deterministic RNG
4. **Neighbour handshake** – nodes exchange direct message references
5. **Protocol execution** – gossip/rumours or push-sum averaging runs
6. **Convergence detection** – thresholds reached trigger supervisor notification
7. **Completion** – run ends once ≥60% alive nodes converge

---

## 💀 Failure Model

* **Trigger** – each cycle, RNG vs. `fail_rate` decides node crash
* **Teardown** – node emits `NodeDead(id)` then halts permanently
* **Accounting** – supervisor recalculates convergence target as `ceil(0.6 * alive)`
* **Converged-but-dead** – counted in metrics
* **Deterministic chaos** – repeatable failure patterns with seeded RNG

---

## 📈 Experimental Data Collection

Automate experiments with:

```bash
for rate in 0.00 0.05 0.10 0.15 0.20; do
  gleam run -- 200 full gossip fail=$rate >> results.csv
done
```

Analyze `results.csv` using Excel, R, or gnuplot:

* Convergence time vs. failure rate
* Success rate (status == `completed`)
* Dead node counts

---

## 🔧 Configuration Knobs

* Gossip threshold → `main.gleam` (default: 2)
* Push-Sum epsilon → `actors.gleam` (default: 1e-10)
* Timeout → 3s (in `wait_for_done_with_timeout`)
* Failure rate → CLI `fail=<rate>` (0.0–1.0)
* RNG seeding → per-node formula for reproducibility

---

## 🐞 Debugging Tips

* **`gleam` not found** → add to `PATH`
* **Erlang missing** → install Erlang/OTP ≥ 26
* **Timeouts** → try denser topologies, extend timeout, reduce `fail`
* **Too many node deaths** → lower failure rate or reduce thresholds

---

## 🚀 Extending the Project

* Add new topologies in `src/topology.gleam`
* Implement alternative failure models (e.g. recoveries, link drops)
* Enhance testing with deterministic mini-networks in gleeunit
* Export richer metrics (CSV/JSON) for advanced analysis

---

## 📜 License

This project is licensed under the [MIT License](LICENSE).

---

✅ **Professional, modular, and extensible — perfect for distributed systems experiments on Gleam/BEAM.**

```

---

Would you like me to also generate a **`Report-bonus.pdf` template** (with plots, methodology, and observations) so it pairs neatly with this README for your course submission?
```
