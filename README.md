# DeepExploit 2

> A research platform for intelligent security decision-making.

**Central research question:**
> Can modern decision-making algorithms select better next actions under uncertainty than static heuristics or random selection?

DeepExploit 2 is **not** a giant exploit collection. It optimises for **better reasoning** about what action should happen next.

---

## Architecture

```
                    ┌───────────────────────┐
                    │  Policy / Scope Gate  │ ← mandatory, no bypass
                    └───────────┬───────────┘
                                │
                    ┌───────────▼───────────┐
                    │      Orchestrator     │ ← depends on contracts only
                    └───────────┬───────────┘
                                │
           Plugin contracts (stable interfaces)
                                │
     ┌──────────────────────────┼──────────────────────────┐
     │                          │                          │
 Tool Plugins           Algorithm Plugins         Observation Plugins
 (NullAdapter,         (Random, Rule,             (Simulated,
  NmapAdapter, …)       Bandit, RL, …)             Nmap, …)
     │                          │                          │
     └──────────────────────────┼──────────────────────────┘
                                │
                    ┌───────────▼───────────┐
                    │   Decision Memory     │ ← SQLite (StoragePlugin)
                    └───────────────────────┘
```

Every action passes through the Policy Gate. STOP is a first-class action.

---

## Quick Start

```powershell
# 1. Install (editable mode with dev extras)
pip install -e ".[dev]"

# 2. Run the example dry-run experiment
deepexploit2 run --experiment experiments/example_experiment.yaml

# 3. Evaluate the run
deepexploit2 evaluate --run-id <run-id-from-output>

# 4. List installed plugins
deepexploit2 plugins

# 5. Run tests
pytest tests/ -v
```

---

## Plugin System

DeepExploit 2 uses a **plugin-first architecture**. Every capability is a plugin:

| Category | Built-in plugins | Purpose |
|---|---|---|
| Tool/Adapter | `null_adapter` | Dry-run simulation |
| Algorithm | `random_algorithm`, `rule_algorithm` | Decision baselines |
| Encoder | `simple_encoder` | Observation → feature vector |
| Observation | `simulated_observation` | Fake but deterministic observations |
| Policy | `scope_policy` | Scope/authorization safety |
| Reward | `default_reward` | Reward signal computation |
| Storage | `sqlite_memory` | SQLite decision memory |
| Evaluation | `default_metrics` | Constitution §10 metrics |

To add an external plugin, create a Python package with:

```toml
[project.entry-points."deepexploit2.plugins"]
my_plugin = "my_package.plugin:MyPlugin"
```

Then `pip install` it. No core modifications required.

---

## Safety

All actions pass through the **PolicyGate** before execution. The gate:
- Calls all active `PolicyPlugin` instances in deterministic order.
- Returns DENY on any failure (fail closed).
- Always permits STOP.
- Requires scope authorization for every target.

**STOP** is a first-class action, not an exception.

---

## Stage Roadmap

| Stage | Status | Focus |
|---|---|---|
| **0** | ✅ Complete | Foundation: domain, plugins, policy, memory, baselines, tests |
| 1 | ⏳ Planned | Nmap observation plugin |
| 2 | ⏳ Planned | Metasploit RPC adapter plugin |
| 3 | ⏳ Planned | Contextual bandit + supervised ranking algorithms |
| 4 | ⏳ Planned | Reinforcement learning algorithm |
| 5 | ⏳ Planned | Security graph reasoning |
| 6 | ⏳ Planned | Explainability layer |
| 7 | ⏳ Planned | Evaluation dashboard |

---

## Project Constitution

This project follows the [DeepExploit 2 Project Constitution](PROMPT_00.md):
- **Research priority**: correctness → observability → reproducibility → decision quality.
- **AI principle**: simplest correct algorithm for the problem type.
- **No RL everywhere**: contextual bandit for single-step, RL for sequential, rules for deterministic.
- **Safety**: fail closed when scope is unknown, authorization is missing, or a security plugin fails.

---

## License

MIT — research use only. Requires explicit authorization for all targets.
