# Automated JIT Logistics & Routing Optimisation
### Digitising Toyota IE Planning Logic into a Python Optimisation Engine

**Role:** Group Head — Logistics Planning Group
**Project Type:** Micro-Logistics Optimisation & Resource Planning

---

> [!IMPORTANT]
> ## Key Result: 40% Fleet Efficiency Gain
> | Metric | Manual Planning | Optimised Engine |
> |---|---|---|
> | Drivers / Tuggers Required | **5** | **3** |
> | Driver Utilisation | Unknown / untracked | **86%** |
> | Total Work Content (per shift) | Estimated manually | **70,079 seconds — mathematically proven** |
> | Planning Method | Iterative, recalculated manually | **Automated — reruns in seconds** |
> | SKUs Managed | 54 parts | 54 parts |
> | Delivery Trips Generated | Manual estimate | **109 synchronised milk-run trips** |
> | Production Lines Covered | 9 line groups | 9 line groups |

---

## What This Project Does — In One Sentence

It takes the complex, time-intensive manual process an Industrial Engineer uses to plan JIT parts delivery in a Toyota assembly plant — and replaces it with a Python engine that produces the same output in seconds, with mathematical proof of the minimum fleet required.

---

## From Physical Plant to Digital Model

The first step was translating the real factory floor into a mathematical representation the optimisation engine could work with.

### Step 1 — The Physical Plant (Source: Toyota Assembly Plant CAD Layout)

This is the actual plant layout the logistics network was designed from — showing assembly lines (Chassis, Trim, Final, Engine), warehouse areas, and the physical aisles that constrain delivery routing.

![Factory CAD Layout](data/Factory_CAD_layout.png)

### Step 2 — The Digitised Factory Graph (NetworkX DiGraph)

Every node (workstation, intersection, staging area) was extracted from the CAD layout and mapped to a coordinate system. Directed edges enforce one-way aisle constraints. Edge weights are real distances in metres. Dijkstra's Algorithm was then applied to find the shortest legal path between every node pair — producing the distance matrix that feeds the optimisation engine.

![Digitised Factory Floor Layout](output/factory_floor_layout_cartesian.png)

> **What you're seeing:** Each circle is a physical location on the shop floor. Arrows show legal one-way travel paths. Numbers are distances in metres. The routing engine can only use paths that exist in this graph — illegal shortcuts are mathematically impossible.

---

## How the Engine Works — 4-Stage Pipeline

```
┌─────────────────────┐     ┌─────────────────────┐     ┌─────────────────────┐     ┌─────────────────────┐
│   STAGE 1           │     │   STAGE 2           │     │   STAGE 3           │     │   STAGE 4           │
│   Spatial Mapping   │────▶│  Workload Modelling │────▶│  Route Optimisation │────▶│  Fleet Sizing       │
│                     │     │                     │     │                     │     │                     │
│ • CAD → coordinates │     │ • 54 SKUs exploded  │     │ • Geographic        │     │ • Total man-seconds │
│ • Node classification│    │   into task list    │     │   clustering        │     │   summed across     │
│ • Directed edges    │     │ • Gentan-i time     │     │ • No duplicate      │     │   450-min shift     │
│   (one-way aisles)  │     │   standards applied │     │   parts per trip    │     │ • Minimum drivers   │
│ • Dijkstra shortest │     │ • Travel + service  │     │ • Traffic conflict  │     │   calculated:       │
│   path → distance   │     │   time calculated   │     │   prevention        │     │   3 drivers @ 86%   │
│   matrix            │     │   per delivery      │     │ • 109 milk-run      │     │   utilisation       │
│                     │     │                     │     │   trips generated   │     │                     │
│ Output: Distance    │     │ Output: Exploded    │     │ Output: Milk Run    │     │ Output: Proven      │
│ Matrix (N×N)        │     │ Task List           │     │ Delivery Groups     │     │ fleet size          │
└─────────────────────┘     └─────────────────────┘     └─────────────────────┘     └─────────────────────┘
```

---

## Stage 3 Output — Sample Milk Run Trips Generated

Each row below is one optimised milk-run trip produced by the engine. The routing path is the exact sequence of nodes the tugger driver follows — derived from the shortest-path distance matrix, respecting all one-way aisle constraints.

| Trip | Line Group | Dollies | Furthest Stop | Routing Path | Travel (s) | Service (s) | Total Cycle (s) |
|------|-----------|---------|--------------|-------------|-----------|------------|----------------|
| TRIP_001 | AC Building | 2 | AC_B | `SML_Dr → I_26 → I_27 → AC_B` | 603.5 | 48 | 651.5 |
| TRIP_006 | Chassis LH | 3 | CL5 | `I_14 → I_15 → EG_L1 → EG_L2 → EG_L3 → CL1 → CL2 → CL3 → CL4 → CL5` | 223.6 | 72 | 295.6 |
| TRIP_026 | Chassis RH | 3 | CR5 | `SML_St → I_21 → I_19 → EG_R1 → EG_R2 → EG_R3 → CR1 → CR2 → CR3 → CR4 → CR5` | 287.4 | 128 | 415.4 |
| TRIP_036 | Engine RH | 2 | EG_R1 | `SML_St → I_21 → I_19 → EG_R1` | 163.8 | 104 | 267.8 |
| TRIP_041 | Final LH | 3 | FL3 | `Bulky_1 → SML_Dr → I_26 → I_27 → I_25 → FL1 → FL2 → FL3` | 649.4 | 128 | 777.4 |

> **Bundling logic applied:** Parts are grouped by shared line-side zone to eliminate redundant travel (Muda). Only the furthest node in a bundle is used to calculate round-trip duration — preventing the distance double-counting common in manual planning. No duplicate parts appear in the same trip, respecting limited rack space at each workstation.

---

## Constraints Engineered Into the System

These are the shop-floor rules that were translated into algorithmic constraints — the same rules an experienced IE planner carries in their head, now enforced mathematically:

| Shop-Floor Rule | How It Was Encoded |
|---|---|
| One-way aisles | Directed edges in the NetworkX DiGraph — illegal paths simply don't exist |
| No duplicate parts per trip | Hard constraint in the bundling algorithm — enforces lineside rack space limits |
| No simultaneous deliveries on same aisle | Traffic conflict detection across all concurrent trips |
| Takt-synchronised delivery frequency | Delivery cycles (10, 50, 100-min) derived from current Takt time |
| Container type determines service time | Gentan-i time standards mapped per SKU container type (Dunnage / Regular Dolly / Custom Dolly) |
| Furthest-node travel time | Single round-trip calculation per bundle — not summed per stop, eliminating double-counting |

---

## Why This Matters Beyond Toyota

The logic in this engine is **industry-agnostic**. The same four-stage pipeline applies to any facility where synchronised point-of-use delivery is required:

- **Mining:** Scheduled parts delivery to drill rigs or processing stations
- **Distribution Centres:** Milk-run replenishment from bulk storage to pick faces
- **Hospitals:** Sterile supplies delivery to operating theatres on timed cycles
- **Truck Body Manufacturing:** Sub-assembly kitting delivery to fabrication workstations

The inputs change. The logic — spatial mapping, workload explosion, route bundling, fleet sizing — stays the same.

---

## Tech Stack

| Tool | Purpose |
|---|---|
| **Python** | Core logic, automation, constraint enforcement |
| **NetworkX** | Factory floor DiGraph, shortest-path (Dijkstra), graph visualisation |
| **Pandas / NumPy** | Data manipulation, task explosion, matrix mathematics |
| **OR-Tools / Custom Bundling** | Constraint-based trip aggregation and traffic conflict detection |

---

## Project Files

| File | Description |
|---|---|
| `factory_floor_layout.py` | Builds the NetworkX DiGraph from node coordinates and directed edges |
| `master_distance_matrix.py` | Runs Dijkstra's Algorithm to generate the full N×N distance matrix |
| `delivery_tasks_list.py` | Explodes demand data into individual work elements with travel + service times |
| `delivery_bundling_levelled_with_traffic_check.py` | Groups tasks into optimised milk-run trips with conflict detection |
| `Factory_CAD_layout.png` | Source plant layout used for node coordinate extraction |
| `factory_floor_layout_cartesian.png` | Digitised factory graph — visual verification of routing logic |
| `Node_coordinates.csv` | (x, y) coordinates for all nodes |
| `From_To.csv` | Directed edge definitions (legal paths) |
| `Distance_Matrix.csv` | N×N shortest-path distance matrix output |
| `Demand.csv` | SKU demand input — 54 parts, delivery frequencies, container types |
| `Exploded_Task_List.csv` | Full task breakdown with routing paths and time calculations |
| `Milk_Run_Delivery_Groups.csv` | Final output — 109 optimised milk-run trips across 9 line groups |

---

> **Confidentiality Note:** This project uses actual Toyota IE planning methodology and operational flow logic. Plant location and specific production model identifiers have been anonymised. Distances, aisle constraints, and service-time variables represent a verified, functional shop-floor environment.

---

[← Back to Main Portfolio](../../README.md)
