Got it 👍 — here’s the same cleaned and professional version formatted properly in **README (Markdown) syntax**:

````markdown
# Gossip & Push-Sum Simulator (Gleam / BEAM)

An end-to-end Gleam application that models the **Gossip** and **Push-Sum** distributed algorithms on top of the **BEAM virtual machine**. Each simulated node runs as a lightweight Erlang process, exchanges messages with its neighbours, and reports convergence to a supervisor that orchestrates the run. The CLI allows you to explore different topologies, inject failures, and export metrics for analysis.

---

## ✨ Features
- Actor-based implementation of Gossip and Push-Sum protocols  
- Built-in topologies: `full`, `line`, `3D`, `imp3D` (3D grid + one random neighbour)  
- Deterministic pseudo-random generator for reproducible experiments  
- Failure injection with `fail=<rate>` to permanently kill nodes (Bernoulli trial)  
- Supervisor dynamically adjusts convergence targets as nodes die and emits CSV metrics  

---

## ⚙️ Requirements
- [Gleam](https://gleam.run) ≥ 1.0  
  (`brew install gleam`, `asdf install gleam`, or binary release)  
- Erlang/OTP ≥ 26 (BEAM runtime)  
- macOS/Linux, or Windows via WSL  
- No additional Hex dependencies beyond `manifest.toml`

Quick toolchain check:
```bash
gleam doctor
````

---

## 📂 Repository Layout

* `src/main.gleam` – CLI entry point, topology bootstrapper, metrics reporter
* `src/actors.gleam` – Gossip/Push-Sum node processes + failure logic
* `src/coordinator.gleam` – Supervisor for convergence & node tracking
* `src/topology.gleam` – Graph builders & statistics for all layouts
* `src/rand_utils.gleam` – Deterministic RNG helpers
* `test/` – Placeholder gleeunit suite

---

## 🛠️ Build & Test

From project root (`project2/`):

```bash
gleam deps download   # Fetch dependencies
gleam build           # Compile project
gleam test            # Run gleeunit tests (placeholder)
```

---

## 🚀 Running Simulations

CLI usage:

```bash
gleam run -- <num_nodes> <topology> <algorithm> [fail=<0.0-1.0>]
```

* `num_nodes` → number of processes
* `topology` → `full`, `line`, `3D`, `imp3D`
* `algorithm` → `gossip` or `push-sum`
* `fail` → optional crash probability (default: 0.0)

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

* `metrics,...` line → CSV-friendly: `num_nodes, topology, algorithm, fail_rate, status, elapsed_ms, dead_nodes, done_nodes, target`
* Final integer → elapsed milliseconds (legacy automation)

---

## 🔎 Simulation Workflow

1. **Topology construction** – build graph, compute degrees
2. **Coordinator bootstrap** – supervisor tracks convergence/deaths
3. **Node spawn** – actors get RNG seed & thresholds
4. **Neighbour handshake** – nodes exchange direct messaging links
5. **Protocol execution** – gossip/rumours or push-sum averaging begins
6. **Convergence detection** – thresholds trigger notifications
7. **Completion** – supervisor ends run at ≥60% alive-node convergence

---

## 💀 Failure Model

* **Trigger** – each cycle, node samples RNG vs. `fail_rate`
* **Teardown** – dead nodes notify supervisor & stop forwarding
* **Accounting** – convergence target recalculated as `ceil(0.6 * alive)`
* **Converged-but-dead** – still counted in metrics
* **Deterministic chaos** – seeds ensure repeatable failure patterns

---

## 📈 Experimental Data Collection

```bash
for rate in 0.00 0.05 0.10 0.15 0.20; do
  gleam run -- 200 full gossip fail=$rate >> results.csv
done
```

Load into plotting tools (`gnuplot`, Excel, R, etc.) → analyze convergence vs. failure rate.

---

## 🔧 Configuration Knobs

* Gossip threshold → `main.gleam` (default = 2)
* Push-Sum epsilon → `actors.gleam` (default = 1e-10)
* Timeout → 3s global cap in `wait_for_done_with_timeout`
* Failure rate → CLI `fail=<rate>`
* RNG seeding → per-node formula for reproducibility

---

## 🐞 Debugging Tips

* **`gleam` not found** → add to PATH
* **Erlang missing** → install Erlang/OTP ≥ 26
* **Timeouts** → use denser topologies, extend timeout, lower failure rate
* **Too many deaths** → reduce `fail` or adjust convergence thresholds

---

## 🚀 Extending the Project

* Add topologies → extend `Topology` + builder in `topology.gleam`
* New failure models → enrich `actors.gleam` & supervisor logic
* Better testing → gleeunit with deterministic mini-networks
* Export more metrics → CSV/JSON for richer analysis

---

✅ **Professional, modular, and extensible — perfect for distributed systems experiments on Gleam/BEAM.**
us, Gleam version, license) at the top so the README looks GitHub-ready?

