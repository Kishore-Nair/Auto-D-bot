# Auto Delivery Bots 🤖🗺️

### Autonomous delivery-route optimization on real-world maps using classical graph algorithms

<hr>

A simulation of autonomous delivery bots navigating **New York Central Park** using real OpenStreetMap road-network data. The project is a hands-on study of core **Data Structures & Algorithms (DSA)** — from shortest-path computation to Traveling Salesman Problem (TSP) approximations — applied to geospatial route planning.

---

## 📂 Project Structure

```
Auto-D-bot/
│
├── server.py                     # Tkinter GUI — user input & delivery orchestration
├── autobot_backend_packages.py   # Core algorithm implementations
├── botmap.html                   # Leaflet.js map — animated route visualization
├── path_coordinates.json         # Generated route data (algorithm output)
├── pkg.png                       # Bot / package icon
├── office.png                    # Sender (start) location icon
└── home.png                      # Destination location icon
```

---

## 🧠 Algorithms & Data Structures

### 1. Graph Construction from Real-World Map Data

The road network of Central Park is modeled as a **weighted, directed graph** `G = (V, E)` using [OSMnx](https://osmnx.readthedocs.io/):

```python
graph = ox.graph_from_place("Central Park, New York, USA", network_type="walk")
```

| Component | Representation |
|---|---|
| **Vertices (V)** | Road intersections (OSM nodes), each with `(lat, lon)` attributes |
| **Edges (E)** | Walkable road segments between intersections |
| **Edge weights** | Physical road length in meters (`weight="length"`) |

The nearest-node lookup (`ox.distance.nearest_nodes`) uses a **Ball Tree / k-d tree** spatial index under the hood for efficient `O(log n)` geospatial queries.

---

### 2. Bellman-Ford Shortest Path Algorithm

**File:** `autobot_backend_packages.py` — `bellman_ford_shortest_path()`

Used for computing the shortest path between two nodes on the road network. This is a custom implementation (not the NetworkX built-in) that handles undirected graphs by relaxing edges in both directions.

**Algorithm summary:**

```
BELLMAN-FORD(G, source, target, weight):
    1. Initialize distance[v] = ∞ for all v ∈ V; distance[source] = 0
    2. Repeat |V| - 1 times:
         For each edge (u, v) with weight w:
           if distance[u] + w < distance[v]:
             distance[v] = distance[u] + w
             predecessor[v] = u
           // Also relax v → u for undirected support
    3. Check for negative-weight cycles (one more pass)
    4. Reconstruct path from predecessor[] array
```

| Property | Value |
|---|---|
| **Time complexity** | `O(V × E)` |
| **Space complexity** | `O(V)` (distance + predecessor arrays) |
| **Negative weights** | ✅ Handled (with cycle detection) |
| **Used in** | `road_small()` — single-destination road delivery |
| | `road_big()` — TSP segment paths on road network |

> **Why Bellman-Ford over Dijkstra?**
> Bellman-Ford was chosen to demonstrate handling of potential negative-weight edges and as a pedagogical exercise. For strictly non-negative road networks, Dijkstra's algorithm (`O((V + E) log V)` with a min-heap) would be more efficient.

---

### 3. Traveling Salesman Problem (TSP) — Library Approximation

**File:** `autobot_backend_packages.py` — `road_big()`

When delivering to **multiple destinations via roads**, the order of visits matters. This is the classic **Traveling Salesman Problem (TSP)**, an NP-hard combinatorial optimization problem.

**Approach — two-phase solution:**

```
PHASE 1 — Build complete distance subgraph:
    For all pairs (i, j) in {start} ∪ {destinations}:
        subgraph[i][j] = shortest_path_length(G, i, j)   // Dijkstra via NetworkX

PHASE 2 — Solve TSP on the subgraph:
    optimal_order = TSP_APPROXIMATION(subgraph, cycle=False)
    // Uses NetworkX's Christofides-based or greedy approximation

PHASE 3 — Reconstruct full route:
    For each consecutive pair in optimal_order:
        segment = bellman_ford_shortest_path(G, source, target)
        Append segment to final route
```

| Property | Value |
|---|---|
| **Problem class** | NP-hard |
| **Approximation used** | `networkx.algorithms.approximation.traveling_salesman_problem` |
| **Approximation ratio** | ≤ 1.5× optimal (Christofides algorithm for metric TSP) |
| **Subgraph construction** | All-pairs shortest path → `O(n² × Dijkstra)` |
| **Used in** | `road_big()` — multi-destination road delivery |

---

### 4. Greedy Nearest Neighbor TSP Heuristic

**File:** `autobot_backend_packages.py` — `air_multiple()` → `tsp_greedy()`

For **aerial (straight-line) multi-destination** delivery, a custom **Greedy Nearest Neighbor** heuristic is implemented:

```
TSP_GREEDY(points):
    route = [start_point]
    unvisited = all points except start
    current = start_point

    while unvisited is not empty:
        nearest = argmin_{p ∈ unvisited} distance(current, p)
        route.append(nearest)
        unvisited.remove(nearest)
        current = nearest

    return route
```

| Property | Value |
|---|---|
| **Time complexity** | `O(n²)` where n = number of destinations |
| **Space complexity** | `O(n)` |
| **Optimality** | Not guaranteed — can be up to `O(log n)` × optimal in worst case |
| **Distance metric** | Euclidean (via `numpy.linalg.norm`) |
| **Used in** | `air_multiple()` — multi-destination aerial delivery |

> **Trade-off:** The greedy heuristic is fast and simple but does not guarantee an optimal tour. For small `n` (≤ 10–15 destinations), it produces acceptable routes. For larger instances, more sophisticated approaches (2-opt improvement, branch-and-bound, or dynamic programming with bitmask — `O(n² × 2ⁿ)`) would yield better results.

---

### 5. Linear Interpolation for Path Smoothing

**File:** `autobot_backend_packages.py` — `interpolate()`

Route segments are densified by inserting evenly-spaced intermediate points for smooth front-end animation:

```
INTERPOLATE(start, end, interval):
    dist = geodesic_distance(start, end)
    num_points = ⌊dist / interval⌋
    for i in 1..num_points:
        lat_i = start.lat + (end.lat - start.lat) × (i × interval / dist)
        lon_i = start.lon + (end.lon - start.lon) × (i × interval / dist)
        yield (lat_i, lon_i)
```

This is standard **linear interpolation** on geographic coordinates, producing intermediate waypoints at a fixed meter-interval spacing.

---

### 6. Geodesic Distance Computation

**File:** `autobot_backend_packages.py` — `totdistance()` & throughout

All real-world distance calculations use the **Vincenty/Haversine geodesic formula** via `geopy.distance.geodesic`, which accounts for Earth's ellipsoidal shape:

```python
from geopy.distance import geodesic
distance_meters = geodesic(point_a, point_b).meters
```

This is used in:
- **Total route distance** accumulation (`totdistance()`)
- **Interpolation** interval calculation
- **Delivery time estimation** (`server.py` — `distance / 5000 m/hr`)

---

## 🔄 Delivery Modes & Algorithm Mapping

| Mode | Function | Pathfinding | Multi-stop Optimization | Distance Metric |
|---|---|---|---|---|
| **Road — Single** | `road_small()` | Bellman-Ford | N/A | Road network (weighted graph) |
| **Road — Multiple** | `road_big()` | Bellman-Ford + NetworkX Dijkstra | TSP approximation (Christofides) | Road network (weighted graph) |
| **Air — Single** | `air_single()` | Direct line | N/A | Geodesic (Haversine/Vincenty) |
| **Air — Multiple** | `air_multiple()` | Direct line | Greedy Nearest Neighbor TSP | Euclidean + Geodesic |

---

## 🏗️ System Architecture

```
┌──────────────────────────┐
│     server.py (Tkinter)  │   User inputs start/end coordinates
│     GUI Controller       │   and selects delivery mode
└──────────┬───────────────┘
           │ calls
           ▼
┌──────────────────────────────────────────────┐
│  autobot_backend_packages.py                 │
│                                              │
│  ┌─────────────┐    ┌──────────────────┐     │
│  │ OSMnx       │───▶│ Graph G = (V, E) │     │
│  │ Map Loader  │    │ NetworkX DiGraph  │     │
│  └─────────────┘    └───────┬──────────┘     │
│                             │                │
│              ┌──────────────┼────────────┐   │
│              ▼              ▼            ▼   │
│    ┌──────────────┐ ┌────────────┐ ┌───────┐│
│    │ Bellman-Ford │ │ TSP Approx │ │Greedy ││
│    │ Shortest Path│ │(Christofid)│ │  NN   ││
│    └──────┬───────┘ └─────┬──────┘ └───┬───┘│
│           └───────┬───────┘            │    │
│                   ▼                    │    │
│           ┌───────────────┐            │    │
│           │  Interpolate  │◀───────────┘    │
│           │  + Geodesic   │                 │
│           └───────┬───────┘                 │
│                   │                         │
└───────────────────┼─────────────────────────┘
                    │ writes JSON
                    ▼
         ┌────────────────────┐
         │path_coordinates.json│
         └─────────┬──────────┘
                   │ fetched by
                   ▼
         ┌────────────────────┐
         │  botmap.html       │   Leaflet.js map with
         │  (Frontend)        │   animated bot marker
         └────────────────────┘
```

---

## 📊 Complexity Summary

| Algorithm | Time | Space | Optimal? |
|---|---|---|---|
| Bellman-Ford | `O(V × E)` | `O(V)` | ✅ Yes (shortest path) |
| TSP — Christofides approx. | `O(n³)` on subgraph | `O(n²)` | ≤ 1.5× optimal |
| TSP — Greedy NN | `O(n²)` | `O(n)` | ❌ Heuristic only |
| Interpolation | `O(d / interval)` per segment | `O(d / interval)` | N/A |
| Geodesic distance | `O(1)` per pair | `O(1)` | ✅ Exact |
| Nearest-node lookup (OSMnx) | `O(log V)` | `O(V)` (tree) | ✅ Exact |

---

## 🚀 How to Run

### Prerequisites

```bash
pip install osmnx networkx geopy numpy
```

### 1. Clone this repo

```bash
git clone https://github.com/Kishore-Nair/Auto-D-bot.git
cd Auto-D-bot
```

### 2. Start a Simple HTTP Server for the Frontend

```bash
python3 -m http.server 8000
```

### 3. Start the Backend Server

In another terminal:

```bash
python3 server.py
```

This opens a Tkinter GUI where you can enter coordinates and select a delivery mode. The computed route is visualized on a Leaflet.js map at `http://localhost:8000/botmap.html`.

> 📍 **Note:** Location data is constrained to **New York Central Park**. Coordinates outside this area will fail to find nearest graph nodes.

---

## 📚 Key Libraries

| Library | Role in Project |
|---|---|
| [OSMnx](https://osmnx.readthedocs.io/) | Downloads & constructs walkable road-network graph from OpenStreetMap |
| [NetworkX](https://networkx.org/) | Graph data structure, shortest-path utilities, TSP approximation |
| [Geopy](https://geopy.readthedocs.io/) | Geodesic distance calculations on WGS-84 ellipsoid |
| [NumPy](https://numpy.org/) | Vector math for Euclidean distance in greedy TSP |
| [Leaflet.js](https://leafletjs.com/) | Interactive map rendering & bot animation in the browser |
| Tkinter | Desktop GUI for user input and delivery orchestration |

---

## 🔮 Possible Improvements

- **Dijkstra's algorithm** — Replace Bellman-Ford with Dijkstra + min-heap for road networks (all-positive weights) → `O((V + E) log V)`
- **A\* search** — Use haversine as an admissible heuristic for faster goal-directed pathfinding
- **2-opt / 3-opt TSP improvement** — Local search post-processing on greedy NN solutions
- **DP with bitmask** — Exact TSP solution for small `n` using `O(n² × 2ⁿ)` dynamic programming
- **Real-time traffic weights** — Dynamic edge weights for time-dependent shortest paths
- **Ant Colony Optimization** — Metaheuristic for larger TSP instances

---

*Built as a study project for Introduction to AI — demonstrating graph algorithms, combinatorial optimization, and geospatial computation on real-world map data.*
