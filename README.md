# Failure Model Deep Dive

This document explains how the Gossip & Push-Sum Simulator implements deterministic node failures, how the supervisor reacts to them, and how to reproduce and analyse a failure-heavy run. Use it as a supplement to the main `README.md`.

## Design Goals
- Provide a configurable crash probability while keeping simulations reproducible.
- Ensure the supervisor dynamically recomputes convergence targets as nodes die.
- Preserve visibility into partially successful runs (converged but dead nodes).
- Support both gossip and push-sum protocols without duplicating logic.

## Implementation Overview
- **Failure sampling** – Each actor evaluates whether it should fail before servicing any new mailbox message. See `src/actors.gleam` where `maybe_trigger_failure/2` samples a deterministic RNG and returns `True` when the drawn float is lower than the configured `failure_rate`.
- **One-shot teardown** – On the first failure, the node notifies the supervisor with `NodeDead(id)` and halts without processing further messages. Already converged nodes skip duplicate reporting because the loop tracks a `done_notified` flag.
- **Supervisor updates** – `src/coordinator.gleam` receives `NodeDead` messages, increments its dead counter, and computes a fresh convergence target using the remaining alive nodes (`alive = total - dead`). The target never falls below `1` while at least one node is alive.
- **Metrics feed** – `src/main.gleam` prints human-readable stats (dead nodes, converged nodes, convergence target) and an aggregated CSV line: `metrics,<num_nodes>,<topology>,<algorithm>,<fail_rate>,<status>,<elapsed_ms>,<dead>,<done>,<target>`.
- **Deterministic randomness** – Each node receives a seeded RNG constructed from its identifier (`id * 7919 + 7`). Re-running with the same topology, `num_nodes`, and `fail` value yields identical crash patterns, ensuring experiments are repeatable.

## Execution Sequence with Failures
1. **Topology bootstrap** – The CLI builds the selected topology and reports connectivity stats.
2. **Coordinator spawn** – A supervisor process starts and publishes its mailbox subject to node actors.
3. **Node bootstrap** – Each node starts with its own RNG, failure rate, convergence thresholds, and supervisor subject.
4. **Failure sampling** – Before each receive loop iteration, the node rolls a float; if the roll is lower than the configured rate, it dies immediately and sends `NodeDead` once.
5. **Protocol progression** – Alive nodes forward gossip rumours or push-sum tuples, seeding new messages during idle ticks to keep the protocol active.
6. **Convergence and death accounting** – The supervisor counts `NodeDone` and `NodeDead` events. When the live convergence threshold is met (60% of surviving nodes), it responds to the CLI with final run statistics.
7. **Metrics emission** – The CLI prints the human-readable block plus the metrics CSV row and raw elapsed time for automation compatibility.

## Reproducing a Failure Scenario
```bash
# Run 20 nodes on a line topology with an aggressive failure rate
cd project2
gleam run -- 20 line gossip fail=0.2
```
Expected highlights:
- Most nodes die early because the failure roll happens every 50 ms.
- The supervisor reduces the convergence target as death notifications arrive.
- Only nodes that converged before crashing contribute to the `done` count.

## Sample Output Explained
```
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
- `Converged target: 1/20` – Only one node needed to converge because 17 deaths reduced the alive population to three.
- `Converged nodes: 1/3` – Exactly one of those three survivors satisfied the gossip threshold.
- `Dead nodes: 17` – Seventeen actors became permanently unavailable because their sampled floats were `< 0.2`.
- `metrics` row – Encodes all critical run statistics in CSV form for downstream parsing.
- Trailing `409` – Supplies elapsed time for legacy tooling expecting a single integer.

## Analytical Workflow
1. **Batch runs** – Use shell loops to run multiple simulations across varying `fail` values and append the metrics lines to a CSV file.
2. **Post-processing** – Import the CSV into your analysis tool (Python, R, spreadsheets) and derive metrics such as average elapsed time, convergence success rate, and death distribution.
3. **Visualisation** – Plot failures versus convergence success to illustrate resilience boundaries. Repeat with different topologies to compare structural robustness.
4. **Reporting** – Summarise methodology, include plots, and reference metrics for reproducibility. The deterministic seeding guarantees others can replicate your results.

## Extending the Failure Model
- **Alternate distributions** – Replace Bernoulli sampling with structured patterns (e.g. clustered failures or time-based ramp-ups) by editing `maybe_trigger_failure/2`.
- **Transient faults** – Introduce a `Recovered` message path to let nodes rejoin after a delay and track recovery statistics in the supervisor.
- **Observability** – Emit richer telemetry (JSON, Prometheus) from `main.gleam` or expose per-node state dumps for debugging.
- **Testing hooks** – Add deterministic unit tests in `test/` that simulate targeted failure sequences and assert on supervisor results.

## Validation Checklist
- `gleam build` – Ensures the simulator compiles after modifications to failure logic.
- `gleam test` – Add and run regression tests covering new failure scenarios.
- Manual runs – Execute targeted simulations (e.g. `fail=0.0`, `fail=0.5`) to validate behaviour under both healthy and degraded conditions.

## Related Files
- `src/actors.gleam` – Failure sampling, gossip/push-sum loops, supervisor notifications.
- `src/coordinator.gleam` – Convergence tracking and target recalculation.
- `src/main.gleam` – CLI wiring, metrics printing, overall orchestration.
- `docs/failure-model.md` – (this document) living reference for the failure subsystem.

Maintain this document when extending the simulator so that advanced users can quickly understand changes to failure handling, reproducibility guarantees, and analysis workflows.
