# Shortest Route Finder: Dijkstra's Algorithm on Indian Cities

A small, dependency-free Python project that models major Indian cities as a weighted graph and uses **Dijkstra's algorithm** to find the shortest road route between any two of them.

By default, it computes the shortest route from **Delhi** to **Chennai**.

## Table of Contents

- [Features](#features)
- [Requirements](#requirements)
- [Quick Start](#quick-start)
- [Example Output](#example-output)
- [How It Works](#how-it-works)
- [Code Structure](#code-structure)
- [Usage Examples](#usage-examples)
- [Customizing the Graph](#customizing-the-graph)
- [Complexity](#complexity)
- [Limitations](#limitations)
- [Possible Improvements](#possible-improvements)

---

## Features

- Undirected weighted graph stored as an adjacency list
- Dijkstra's algorithm using a binary min-heap (`heapq`)
- Early exit once the destination is settled
- Path reconstruction, not just distance
- Graceful handling of unreachable cities
- Pure Python standard library, no installs needed

## Requirements

- Python 3.6+

## Quick Start

```bash
git clone https://github.com/<your-username>/<your-repo>.git
cd <your-repo>
python route.py
```

## Example Output

```
Shortest path from Delhi to Chennai:
Delhi -> Agra -> Gwalior -> Bhopal -> Nagpur -> Hyderabad -> Chennai
Total distance: 2258 km
```

Breakdown of the route:

| Leg | Distance (km) | Cumulative (km) |
|---|---:|---:|
| Delhi → Agra | 233 | 233 |
| Agra → Gwalior | 120 | 353 |
| Gwalior → Bhopal | 425 | 778 |
| Bhopal → Nagpur | 350 | 1128 |
| Nagpur → Hyderabad | 500 | 1628 |
| Hyderabad → Chennai | 630 | 2258 |

## How It Works

Dijkstra's algorithm finds the lowest-cost path from a source node to all other nodes in a graph with **non-negative edge weights**. Here, nodes are cities and edge weights are road distances in kilometres.

1. Set the distance to the start city to `0`; all others are implicitly infinity.
2. Push `(0, start)` onto a min-heap.
3. Repeatedly pop the city with the smallest known distance.
4. If it was already visited, skip it. Otherwise mark it visited.
5. If it is the goal, stop.
6. For each neighbor, compute `dist + weight`. If this beats the best known distance, update it, record the current city in `previous`, and push the neighbor onto the heap.
7. After the loop, rebuild the route by walking backwards through `previous` from the goal to the start.

Because the heap may contain stale entries (a city pushed multiple times with different distances), the `visited` check on pop discards outdated ones. This is the standard "lazy deletion" approach.

## Code Structure

Everything lives in a single file, `route.py`.

### `class Graph`

An undirected weighted graph backed by a dictionary of dictionaries.

| Method | Description |
|---|---|
| `add_node(name)` | Adds a city if it does not already exist. |
| `add_edge(city1, city2, distance)` | Adds a bidirectional road between two cities, creating nodes as needed. |
| `neighbors(city)` | Returns a `{neighbor: distance}` dict for a city (empty if unknown). |

Internal representation:

```python
{
    "Delhi":  {"Jaipur": 280, "Agra": 233, "Kanpur": 440, "Chandigarh": 250},
    "Jaipur": {"Delhi": 280, "Ahmedabad": 660, "Udaipur": 395},
    ...
}
```

### `dijkstra(graph, start, goal)`

Returns a tuple `(path, total_distance)`.

| Return value | Meaning |
|---|---|
| `path` | List of city names from `start` to `goal`, or `None` if unreachable |
| `total_distance` | Total distance in km, or `float('inf')` if unreachable |

### Graph data

The `edges` list near the bottom of the file holds every road as `(city1, city2, distance_km)` and is loaded into a global `Graph` instance `g`.

### Entry point

When run as a script, the code solves `Delhi → Chennai` and prints the result. The `if __name__ == "__main__":` guard means you can safely `import route` from other modules without triggering the output.

## Usage Examples

### Change the start and destination

Edit the line in the `__main__` block:

```python
start, goal = "Mumbai", "Kolkata"
```

### Use it as a module

```python
from route import g, dijkstra

path, distance = dijkstra(g, "Amritsar", "Mysore")
print(path)       # list of cities
print(distance)   # total km
```

### Handle unreachable cities

```python
from route import Graph, dijkstra

island = Graph()
island.add_edge("A", "B", 10)
island.add_node("C")  # isolated

path, dist = dijkstra(island, "A", "C")
print(path, dist)  # None inf
```

> **Note:** City names are case-sensitive and must match the graph exactly, including the spelling `Vishakhapatnam` used in the data. Passing a `start` city that is not in the graph will raise an error when reconstructing the path, so validate input if you accept user-provided names.

## Customizing the Graph

Add new cities or roads by appending to the `edges` list:

```python
edges = [
    ...,
    ("Jaipur", "Jodhpur", 340),
    ("Hyderabad", "Warangal", 150),
]
```

New cities are created automatically. Because the graph is undirected, you only need to declare each road once.

## Cities Included

Amritsar, Chandigarh, Delhi, Jaipur, Agra, Kanpur, Lucknow, Varanasi, Patna, Gwalior, Bhopal, Indore, Udaipur, Ahmedabad, Surat, Mumbai, Pune, Nashik, Aurangabad, Solapur, Nagpur, Raipur, Hyderabad, Vishakhapatnam, Vijayawada, Kolkata, Bhubaneswar, Bangalore, Chennai, Mysore (30 cities, 47 roads).

## Complexity

With `V` cities and `E` roads:

| | Complexity |
|---|---|
| Time | `O((V + E) log V)` |
| Space | `O(V + E)` |

## Limitations

- **Illustrative data:** distances are simplified and approximate. They are not suitable for real navigation.
- **Non-negative weights only:** Dijkstra's algorithm does not work with negative edge weights.
- **Distance only:** travel time, tolls, road quality, and traffic are not modeled.
- **Hard-coded graph:** the data is embedded in the script rather than loaded from a file.

## Possible Improvements

- Accept `start` and `goal` via command-line arguments (`argparse`)
- Load the graph from a JSON or CSV file
- Validate city names and suggest close matches
- Return the top *k* shortest routes
- Add A* search with straight-line distance as a heuristic
- Add unit tests with `pytest`
- Visualize the computed route on the map image

## Acknowledgements

The map is modeled on the Romania road map example (Figure 3.3) from *Artificial Intelligence: A Modern Approach* (AIMA) by Russell and Norvig, adapted here to Indian cities.

