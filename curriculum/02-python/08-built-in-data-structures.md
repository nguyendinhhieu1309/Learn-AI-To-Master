# Built-in Data Structures and Their Costs

Choose a structure based on the operations the program performs. Python's built-ins are optimized and should usually be preferred over a handwritten replacement.

## 1. List: a dynamic array

A list stores an ordered sequence and supports indexing.

| Operation | Typical complexity |
| --- | --- |
| Read or assign by index | O(1) |
| Search for a value | O(n) |
| Append at end | amortized O(1) |
| Insert or remove near front | O(n), due to shifting |
| Sort | O(n log n) worst case |

Use a list for batches, ordered results, stacks, and data that needs index access.

## 2. Tuple and range

A tuple is an immutable ordered record or sequence. A range represents an integer progression compactly without materializing all values.

~~~python
point = (3.5, 7.2)
for index in range(1_000_000):
    pass
~~~

Immutability makes tuples useful as dictionary keys when every member is hashable.

## 3. Dictionary and set

Dictionary and set lookup, insertion, and deletion are average-case O(1), with O(n) worst cases under pathological collisions. A dictionary maps keys to values; a set stores only keys.

~~~python
counts = {}
for label in ["cat", "dog", "cat"]:
    counts[label] = counts.get(label, 0) + 1
~~~

Use a set for membership and uniqueness. Keys need stable equality and hashing. Do not mutate an object in a way that changes its hash while it is stored as a key.

## 4. Queue: use deque

Removing the first element of a list shifts remaining elements and costs O(n). Use collections.deque for efficient operations at both ends:

~~~python
from collections import deque

queue = deque(["first"])
queue.append("second")
item = queue.popleft()
~~~

Appending or removing at either end is O(1). Deques do not provide efficient arbitrary indexing in the middle.

## 5. Priority queue: use heapq

A min-heap exposes the smallest item. Heap insertion and removal cost O(log n); reading the smallest item costs O(1).

~~~python
import heapq

tasks = [(4, "later"), (1, "urgent"), (3, "normal")]
heapq.heapify(tasks)
priority, task = heapq.heappop(tasks)
~~~

Use a priority queue when the next item should be selected by priority rather than insertion order.

## 6. Choose by required operation

| Need | Good starting point |
| --- | --- |
| Ordered sequence and random index | list |
| Keyed lookup | dict |
| Fast membership and unique items | set |
| Add/remove at both ends | deque |
| Always take minimum or maximum priority | heapq |
| Fixed immutable grouping | tuple |

The fastest asymptotic choice may not be fastest for tiny data. Prefer clear code, then measure if performance matters.

## Practice with answers

1. Why is removing index zero from a list O(n)?
2. Which structure should implement the waiting line in BFS?
3. Which structure finds the next smallest scheduled event?

### Answers

1. Remaining elements shift left.
2. deque.
3. A min-heap.
