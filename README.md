# Intelligent Warehouse Optimizer

This project simulates a workflow in a manufacturing and warehouse environment where the sales department inputs customer orders into the system to automatically generate optimized routes for a forklift, check stock levels against safety stock for replenishment if needed, and return a trip document via API for further processing.

---

## 📋 Project Overview

The **Intelligent Warehouse Optimizer** combines inventory management (Reorder Point / Safety Stock) with vehicle routing optimization (VRP) to deliver an end-to-end warehouse operations tool. The pipeline covers data preparation, graph-based pathfinding, constrained optimization, and a live Streamlit dashboard.

---

## 🗺️ Development Roadmap

### Week 1 — Data Preparation & Statistical Foundation

**Goal:** Build a clean, enriched dataset and identify which items are at risk of stockout.

| Step | Description |
|------|-------------|
| Data Prep | Load the Supply Chain dataset. Add `unit_weight` (randomized) and `x`, `y` coordinates for each `location_id`. |
| RFM Segmentation | Categorize items into **A** (Critical), **B** (Medium), and **C** (Low) priority using Python. |
| Statistical Audit | Calculate Dynamic Safety Stock using the Z-score formula: $SS = Z \times \sigma_d \times \sqrt{L}$, where $Z = 1.96$ for 97.5% service level, $\sigma_d$ is the standard deviation of daily demand, and $L$ is the lead time in days. |
| EDA | Create charts in Python / Tableau identifying items where `Stock < Safety Stock` ("At Risk"). |

---

### Week 2 — Graph & Routing Logic (The "Brain")

**Goal:** Build a warehouse graph and compute the shortest legal path between every pair of locations.

| Step | Description |
|------|-------------|
| Graph Construction | Define warehouse **Nodes** (aisle intersections) and **Edges** (aisles) with real distances. |
| Pathfinding (Dijkstra) | Use the **NetworkX** library to run Dijkstra's algorithm between all location pairs. |
| Distance Matrix | Generate a table storing the shortest "legal" distance between every possible pair of locations — the input consumed by the VRP in Week 3. |

```python
import networkx as nx

G = nx.Graph()
G.add_weighted_edges_from(edges)  # edges = [(node_a, node_b, distance), ...]

distance_matrix = dict(nx.all_pairs_dijkstra_path_length(G))
```

---

### Week 3 — The Optimizer (VRP & Constraints)

**Goal:** Solve the Vehicle Routing Problem (VRP) to minimize total forklift travel while respecting capacity.

**Model Formulation** (PuLP / Google OR-Tools)

* **Objective:** Minimize total travel distance across all forklift trips.
* **Constraint 1 — Capacity:** Total weight per trip ≤ 1,000 kg (forklift limit).
* **Constraint 2 — Coverage:** Every item in the Sales Order must be picked exactly once.

**Heuristic Fallback:** If the exact VRP solver is too slow, a **Greedy Nearest-Neighbor** algorithm selects the closest unpicked location that still fits within the remaining weight capacity of the current trip.

```python
# Greedy nearest-neighbor sketch
def greedy_route(orders, distance_matrix, capacity=1000):
    route, load = [], 0
    current = "START"
    remaining = orders.copy()
    while remaining:
        nearest = min(remaining, key=lambda o: distance_matrix[current][o.location])
        if load + nearest.weight <= capacity:
            route.append(nearest)
            load += nearest.weight
            remaining.remove(nearest)
            current = nearest.location
        else:
            route.append("RETURN_TO_BASE")
            load, current = 0, "START"
    return route
```

---

### Week 4 — Interface & Deployment

**Goal:** Deliver a LinkedIn-ready Streamlit dashboard that non-technical warehouse staff can operate.

#### Streamlit UI Features

1. **Sales Order Input** — Select or upload a list of items to pick.
2. **Optimize Button** — Trigger the VRP / greedy solver.
3. **Route Map** — Visual overlay of the forklift path on the warehouse floor plan.
4. **Trip Paper Generator** — Printable summary listing items in pick order with location, quantity, and weight.

```bash
streamlit run app.py
```

---

## 🛠️ Tech Stack

| Layer | Tools |
|-------|-------|
| Data & Statistics | Python, Pandas, NumPy, SciPy |
| Visualization | Matplotlib, Seaborn, Plotly, Tableau |
| Graph / Routing | NetworkX (Dijkstra) |
| Optimization | PuLP / Google OR-Tools |
| UI | Streamlit |
| Version Control | Git / GitHub |

---

## 📐 Key Formula — Dynamic Safety Stock

$$SS = Z \times \sigma_d \times \sqrt{L}$$

| Symbol | Meaning |
|--------|---------|
| $Z$ | Service-level Z-score (e.g., 1.96 for 97.5%) |
| $\sigma_d$ | Standard deviation of daily demand |
| $L$ | Lead time (days) |

**Reorder Point (ROP):**

$$ROP = \bar{d} \times L + SS$$

where $\bar{d}$ is the average daily demand.

---

## 🎓 Courses Applied

| Course | Application |
|--------|-------------|
| **ADS** — Algorithms & Data Structures | Graph construction, Dijkstra pathfinding |
| **DAA** — Design & Analysis of Algorithms | VRP formulation, greedy heuristic |
| **Opt** — Operations Research / Optimization | Linear programming model (PuLP / OR-Tools), safety-stock calculations |

---

## 🚀 Getting Started

```bash
# 1. Clone the repository
git clone https://github.com/billythanapong/Inventory-ROP-and-Routing-Optimization.git
cd Inventory-ROP-and-Routing-Optimization

# 2. Install dependencies
pip install -r requirements.txt

# 3. Run the Streamlit app
streamlit run app.py
```

---

## 📁 Project Structure (planned)

```
├── data/
│   └── supply_chain.csv          # Raw supply chain dataset
├── notebooks/
│   ├── week1_eda.ipynb            # RFM segmentation & safety stock
│   ├── week2_graph.ipynb          # Graph construction & distance matrix
│   └── week3_vrp.ipynb            # VRP formulation & heuristic
├── src/
│   ├── safety_stock.py            # Safety stock & ROP calculations
│   ├── graph.py                   # Warehouse graph & Dijkstra
│   ├── vrp.py                     # VRP solver & greedy fallback
│   └── trip_paper.py              # Trip document generator
├── app.py                         # Streamlit dashboard
├── requirements.txt
└── README.md
```

---

## 📄 License

This project is released under the [MIT License](LICENSE).
