# Graph và thuật toán graph

[Xem bài gốc bằng tiếng Anh](../../../../curriculum/02-python/12-graphs-and-graph-algorithms.md)

Graph mô hình hóa các mối quan hệ: thành phố và con đường, người dùng và kết nối, tác vụ và dependency, hoặc trạng thái và chuyển tiếp.

## Mục tiêu học tập

- Biểu diễn graph có hướng, vô hướng, có trọng số và không trọng số.
- Cài đặt breadth-first search (BFS) và depth-first search (DFS).
- Tìm đường đi ngắn nhất trên graph không trọng số và graph có trọng số không âm.
- Phân tích thuật toán graph theo số đỉnh V và số cạnh E.

## 1. Thuật ngữ và cách biểu diễn graph

Graph gồm vertex (node) và edge. Edge có thể có hướng hoặc vô hướng, có trọng số hoặc không trọng số. Path là chuỗi các vertex được nối với nhau; cycle quay trở lại một vertex đã gặp trước đó.

Adjacency list:

~~~python
graph = {
    "A": ["B", "C"],
    "B": ["A", "D"],
    "C": ["A"],
    "D": ["B"],
}
~~~

Adjacency list dùng O(V+E) bộ nhớ và là lựa chọn mặc định tốt cho graph thưa. Adjacency matrix dùng O(V^2) bộ nhớ và kiểm tra edge trong O(1), phù hợp với graph dày.

## 2. Breadth-first search

BFS duyệt theo từng lớp khoảng cách bằng queue. Nó tìm được path ngắn nhất trong graph không trọng số.

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

Đánh dấu node đã thăm ngay khi đưa vào queue; nếu đợi đến lúc lấy ra thì cycle có thể gây lặp công việc. Với adjacency list, BFS tốn O(V+E) thời gian và O(V) bộ nhớ phụ.

## 3. Depth-first search

DFS đi sâu theo một nhánh trước khi quay lui. Nó hỗ trợ kiểm tra khả năng đi tới, tìm connected component, phát hiện cycle và sắp xếp topo.

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

Implementation lặp này tránh giới hạn độ sâu đệ quy của Python. Muốn duyệt tất cả component, hãy đi qua mọi vertex và bắt đầu traversal ở mỗi node chưa thăm.

## 4. Đường đi ngắn nhất bằng Dijkstra

Dijkstra tìm đường đi ngắn nhất từ một nguồn khi trọng số mọi edge không âm. Thuật toán dùng min-heap cho các khoảng cách đang xét.

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

Function trả về khoảng cách và liên kết predecessor; dùng reconstruct_path để tạo route. Target không thể tới sẽ có khoảng cách vô cực và không có route. Với binary heap và adjacency list, complexity là O((V+E) log V). Dùng Bellman-Ford khi cho phép trọng số âm; thuật toán này phát hiện được negative cycle nhưng chậm hơn. Nếu mọi edge có cùng cost, BFS đơn giản và nhanh hơn.

## 5. Tác vụ graph thường gặp

- Reachability: có path nào từ source đến target không?
- Connected component: trong graph vô hướng, nhóm vertex nào nối được với nhau?
- Cycle detection: theo quy tắc tương ứng của graph có hướng/vô hướng, path có quay lại node đang hoạt động hoặc đã thăm không?
- Topological order: sắp xếp directed acyclic graph (DAG) sao cho prerequisite đứng trước.
- Shortest path: tối thiểu số edge với graph không trọng số hoặc tổng trọng số với graph có trọng số.

Hãy xác định rõ vertex không tồn tại sẽ được thêm, bỏ qua hay báo lỗi. Nêu giá trị cần trả về nếu không có path.

## Bài luyện tập kèm đáp án

1. Search nào tìm path có ít edge nhất trong graph không trọng số?
2. Vì sao Dijkstra phải từ chối trọng số âm?
3. Bộ nhớ adjacency list và chi phí traversal là bao nhiêu?
4. BFS tránh thăm lại node bằng cách nào?

### Đáp án

1. BFS.
2. Edge âm xuất hiện về sau có thể cải thiện khoảng cách node đã chốt, làm sai bước tham lam của Dijkstra.
3. Bộ nhớ O(V+E), traversal O(V+E).
4. Ghi nhận node đã khám phá ngay khi đưa vào queue.
