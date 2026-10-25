# Graphs and Graph Algorithms

A graph models relationships: cities and roads, users and connections, tasks and dependencies, or states and transitions.

## Learning objectives

- Represent directed, undirected, weighted, and unweighted graphs.
- Implement breadth-first search (BFS) and depth-first search (DFS).
- Find shortest paths in unweighted graphs and non-negative weighted graphs.
- Analyze graph algorithms using vertices V and edges E.

## 1. Graph vocabulary and representation

A graph has vertices (nodes) and edges. Edges may be directed or undirected, weighted or unweighted. A path is a sequence of connected vertices; a cycle returns to an earlier vertex.

Adjacency list:

~~~python
graph = {
    "A": ["B", "C"],
    "B": ["A", "D"],
    "C": ["A"],
    "D": ["B"],
}
~~~

An adjacency list uses O(V+E) space and is a good default for sparse graphs. An adjacency matrix uses O(V^2) space and makes edge-existence checks O(1), useful for dense graphs.

## 2. Breadth-first search

BFS explores by distance layers using a queue. It finds a shortest path in an unweighted graph.

~~~python
from collections import deque

def bfs_distances(graph: dict[str, list[str]], start: str) -> dict[str, int]:
    distance = {start: 0}
    queue = deque([start])
    while queue:
        node = queue.popleft()
        for neighbor in graph.get(node, []):
            if neighbor not in distance:
                distance[neighbor] = distance[node] + 1
                queue.append(neighbor)
    return distance
~~~

Mark a node visited when it is enqueued; otherwise cycles can cause repeated work. With adjacency lists, BFS costs O(V+E) time and O(V) extra space.

## 3. Depth-first search

DFS follows one branch deeply before backtracking. It supports reachability, connected components, cycle detection, and topological sorting.

~~~python
def dfs_order(graph: dict[str, list[str]], start: str) -> list[str]:
    visited = set()
    order = []
    stack = [start]
    while stack:
        node = stack.pop()
        if node in visited:
            continue
        visited.add(node)
        order.append(node)
        for neighbor in reversed(graph.get(node, [])):
            if neighbor not in visited:
                stack.append(neighbor)
    return order
~~~

This iterative version avoids Python's recursion-depth limit. DFS over all components loops over all vertices and starts a traversal for every unseen vertex.

## 4. Dijkstra's shortest path

Dijkstra finds shortest paths from one source when every edge weight is non-negative. It uses a min-heap of candidate distances.

~~~python
import heapq
from math import inf

def dijkstra(graph: dict[str, list[tuple[str, float]]], start: str):
    nodes = set(graph)
    for edges in graph.values():
        for neighbor, weight in edges:
            if weight < 0:
                raise ValueError("Dijkstra requires non-negative weights")
            nodes.add(neighbor)

    distance = {node: inf for node in nodes}
    distance[start] = 0
    previous = {}
    heap = [(0, start)]

    while heap:
        current_distance, node = heapq.heappop(heap)
        if current_distance != distance[node]:
            continue
        for neighbor, weight in graph.get(node, []):
            candidate = current_distance + weight
            if candidate < distance.get(neighbor, inf):
                distance[neighbor] = candidate
                previous[neighbor] = node
                heapq.heappush(heap, (candidate, neighbor))
    return distance, previous

def reconstruct_path(previous: dict[str, str], start: str, target: str):
    if target != start and target not in previous:
        return None
    path = [target]
    while path[-1] != start:
        path.append(previous[path[-1]])
    path.reverse()
    return path
~~~

The function returns distances and predecessor links; use reconstruct_path to produce a route. An unreachable target has infinite distance and no route. With a binary heap and adjacency list, complexity is O((V+E) log V). Use Bellman-Ford when negative edge weights are allowed; it can detect negative cycles but is slower. BFS is simpler and faster when every edge has equal cost.

## 5. Common graph tasks

- Reachability: is there any path from source to target?
- Connected components: which vertices are mutually reachable in an undirected graph?
- Cycle detection: does a path return to an active or visited vertex under the relevant directed/undirected rules?
- Topological order: order a directed acyclic graph (DAG) so each prerequisite comes first.
- Shortest path: minimize edge count for unweighted graphs or total weight for weighted graphs.

Clarify whether missing vertices should be added, ignored, or rejected. State what to return when no path exists.

## Practice with answers

1. Which search gives the fewest-edge path in an unweighted graph?
2. Why must Dijkstra reject negative weights?
3. What are the adjacency-list space and traversal costs?
4. How does BFS avoid revisiting nodes?

### Answers

1. BFS.
2. A later negative edge can improve a node already finalized, invalidating Dijkstra's greedy step.
3. O(V+E) storage and O(V+E) traversal.
4. Record a node as visited/discovered when enqueuing it.
