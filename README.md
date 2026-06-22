# Toy Traffic Simulator

A discrete-event traffic network simulator written in Python, built for **Communication Networks (Assignment 6)**. Supports arbitrary road topologies defined in JSON, multiple vehicle classes with priority-weighted routing, adaptive traffic signal control, and a built-in GUI network editor.

---

## Features

- **JSON-defined network topologies** — nodes, roads, lanes, speed limits, and source/sink configuration all in one file
- **Multi-class vehicle routing** — each vehicle type carries a destination and priority weight; routing uses Dijkstra's algorithm over the road graph
- **Weighted Fair Queuing (WFQ) traffic signals** — junctions serve queued vehicles by class weight; green time adapts per-lane demand within configurable min/max bounds
- **Poisson arrival process** — vehicle arrivals at source nodes modelled as Poisson processes with per-class flow rates
- **Real-time visualiser** — animated simulation view with live junction queue lengths and vehicle positions
- **Built-in network editor** — drag-and-drop GUI (`--edit` flag) to construct or modify topologies without editing JSON by hand
- **Statistics dashboard** — per-junction road utilisation, arrival rate histograms, and queue-length bar charts exported at end of run

---

## Repository Structure

```
Toy-Traffic-Simulator/
├── main.py                      # Entry point — simulation and editor modes
├── engine.py                    # Discrete-event simulation core
├── junction.py                  # Junction logic: WFQ signal control
├── road.py                      # Road / lane model
├── vehicle.py                   # Vehicle state and class attributes
├── router.py                    # Dijkstra-based shortest path routing
├── router_2.py                  # Alternate routing variant (used in Experiment 2)
├── source_sink.py               # Poisson arrival / departure logic
├── visualiser.py                # Pygame real-time animation
├── netedit.py                   # Network editor GUI
├── network.json                 # Network topology 1 — regular grid (12 nodes)
├── network2.json                # Network topology 2 — asymmetric hub-spoke (10 nodes)
├── city_10_node_demo.json       # Legacy reference topology
├── outputs/
│   ├── simulation.gif           # Topology 1 — WFQ with min-green cap (10 s)
│   ├── simulation2.gif          # Topology 2 — WFQ with min-green cap (10 s)
│   ├── statistics.png           # Dashboard: Topology 1
│   ├── statistics2.png          # Dashboard: Topology 2
│   ├── stats.json               # Raw stats: Topology 1
│   ├── stats2.json              # Raw stats: Topology 2
│   ├── stats_summary.md         # Human-readable summary: Topology 1
│   ├── stats_summary2.md        # Human-readable summary: Topology 2
│   └── wfq_uncapped/            # Experiment: WFQ without minimum green time floor
│       ├── simulation.gif
│       ├── statistics.png
│       ├── stats.json
│       └── README.md            # Experiment notes and result comparison
└── requirements.txt
```

---

## Getting Started

### Install dependencies

```bash
pip install -r requirements.txt
```

### Run a simulation

```bash
python main.py --network network.json --duration 300
```

Options:

| Flag | Description | Default |
|---|---|---|
| `--network` | Path to network JSON file | `network.json` |
| `--duration` | Simulation duration in seconds | `300` |
| `--no-vis` | Headless mode (no Pygame window) | off |

### Open the network editor

```bash
python main.py --edit
```

Use the editor to place junctions, draw roads, set lane counts and speed limits, configure source/sink nodes, and export to JSON. No JSON editing required.
<img width="1176" height="699" alt="image" src="https://github.com/user-attachments/assets/f12f08e7-6f6d-4860-b735-31ac03d62917" />

---

## Network Format

Topologies are defined in JSON with two top-level arrays: `nodes` and `roads`.

**Node fields (excerpt):**

```json
{
  "id": "J1",
  "x": 235, "y": 208,
  "is_source": true,
  "source_rate": 0.5,
  "source_mode": "poisson",
  "destinations": ["J5", "J6"],
  "vehicle_types": [
    { "type_id": 1, "destination": "J5", "flow_rate": 0.3, "weight": 1, "color": "#e53935" }
  ],
  "signal_algorithm": "wfq_lane",
  "min_green": 4.0,
  "max_green": 12.0,
  "service_rate": 1
}
```

**Road fields (excerpt):**

```json
{
  "id": "R1A",
  "from": "J1", "to": "J7",
  "length": 200,
  "speed_limit": 12,
  "lanes": 1,
  "corridor_id": "C1"
}
```

Roads are directional — bidirectional corridors use paired `A`/`B` road IDs.

---

## Experiments

Two network topologies were tested, each under two signal control configurations.

### Topology 1 — Regular grid (`network.json`)

12 nodes in a 4×3 grid, uniform corridor lengths. 4 source nodes, up to 4 vehicle classes per source.

### Topology 2 — Asymmetric hub-spoke (`network2.json`)

10 nodes, irregular layout, 4 source nodes generating 5 vehicle classes. High-demand corridor J1 → J5 → J6.


### Signal control variants

| Variant | Min green cap | Notes |
|---|---|---|
| WFQ with cap | 10 s | Prevents starvation at lightly-loaded junctions |
| WFQ uncapped (`outputs/wfq_uncapped/`) | None | Allocates time purely by queue weight — better throughput observed on both topologies |

The uncapped variant consistently achieved lower mean wait times and higher arrived-vehicle counts over a 300 s run. See `outputs/wfq_uncapped/README.md` for the full comparison.

---

## Results (Topology 1, WFQ uncapped, 300 s)

| Metric | Value |
|---|---|
| Vehicles created | 134 |
| Vehicles arrived | 103 |
| Average wait time | ~30 s |
| Throughput | ~0.34 veh/s |

*(Update these figures from `stats.json` after re-running with your final parameters.)*

---

## Screenshots

### Simulation (Topology 1)
![Simulation GIF](outputs/simulation.gif)

### Simulation (Topology 2)

![Simulation GIF](outputs/simulation2.gif)

### Network Editor


<img width="1171" height="679" alt="image" src="https://github.com/user-attachments/assets/256b53c3-02f3-4a54-b050-216461130d43" />

### Statistics Dashboard

![Statistics Dashboard](outputs/statistics.png)

---

## Signal Algorithm: WFQ Lane

At each junction, incoming vehicles are grouped by class (vehicle type × destination). The WFQ scheduler allocates green time proportional to `queue_length × weight` for each class, bounded by `[min_green, max_green]`. This approximates weighted fair queuing from packet scheduling adapted to discrete-time traffic flow.

---

## Dependencies

- Python 3.9+
- `pygame` — visualiser and editor
- `matplotlib` — statistics dashboard
- `networkx` — graph construction for Dijkstra routing

Full list in `requirements.txt`.

---
