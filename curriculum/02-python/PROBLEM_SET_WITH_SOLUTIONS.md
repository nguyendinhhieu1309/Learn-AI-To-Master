# Python, OOP, and DSA: Problem Set with Solutions

Work through the questions in order. For algorithm questions, state assumptions and analyze both time and extra space.

## 1. Slicing

What does values[1:4] return for values=[10,20,30,40,50]?

### Solution

[20,30,40]. The start is included and the stop is excluded.

## 2. Aliasing

Two names point to the same list. One name appends an item. Does the other see it?

### Solution

Yes. Assignment binds a second name to the same mutable object. Use a copy when an independent list is needed.

## 3. Count frequencies

Give a structure and complexity for counting repeated labels in n records.

### Solution

Use a dictionary from label to count. Expected time is O(n), extra storage O(k), where k is the number of distinct labels.

## 4. Mutable default

Why is def add(x, items=[]) risky, and what pattern avoids shared state?

### Solution

The default list is created once and reused across calls. Use items=None, then create a new list inside the function when items is None.

## 5. Return or print

A normalization function prints transformed values. Why is that harder to reuse than returning them?

### Solution

Printed output is presentation and is difficult for another function to process. Returning values supports composition, testing, and saving in different formats.

## 6. OOP or function?

A function computes IoU for two boxes and stores no state. Must it be a class?

### Solution

No. A pure function is simpler. Use a class when meaningful state and behavior belong together, or a data class to represent a box.

## 7. Inheritance or composition?

A detector uses a replaceable image preprocessor. Should the detector inherit from the preprocessor?

### Solution

Usually no. A detector has a preprocessor, so composition expresses the relationship and makes the component replaceable.

## 8. Complexity of separate loops

One loop processes n rows; a second loop processes m columns. What is total time?

### Solution

O(n+m), not O(nm), because the loops run sequentially.

## 9. Complexity of pair comparisons

A nested loop compares every pair from an n-element list. What is its time complexity?

### Solution

O(n^2). It may do n squared comparisons, or n(n-1)/2 if comparing unordered distinct pairs; both simplify to O(n^2).

## 10. Linear versus binary search

A list has one million sorted values. What is the asymptotic comparison cost of linear and binary search?

### Solution

Linear search is O(n), up to about one million checks. Binary search is O(log2 n), about 20 halving steps. Binary search requires sorted random-access data.

## 11. List or deque queue

Why is repeated list.pop(0) a poor queue implementation?

### Solution

Each removal shifts the remaining elements, costing O(n). deque.popleft is O(1) at the end.

## 12. Linked-list complexity

What is append complexity with and without a stored tail pointer?

### Solution

With a tail pointer, O(1); without it, O(n) traversal to the last node. Search by value is O(n) in either case.

## 13. Hash table costs

What are expected and worst-case dictionary lookup complexities?

### Solution

Expected/average O(1), worst case O(n). Hashing does not guarantee constant time for every possible collision pattern.

## 14. Stable sorting

Two records have the same score. What does stable sorting preserve?

### Solution

Their relative order from the input. This is useful for multi-key sorting performed in stages.

## 15. BST degeneration

Insert values 1,2,3,4 in that order into an unbalanced BST. What shape and search complexity can result?

### Solution

A chain leaning right. Height is O(n), so search and insertion can degrade to O(n).

## 16. BST deletion

How do you delete a BST node with two children?

### Solution

Replace it with its in-order successor or predecessor, then remove that replacement from its original position. The replacement has at most one child.

## 17. Heap versus sorted list

You repeatedly add tasks and remove the smallest priority. Why can a heap be preferable?

### Solution

Heap insertion and removal are O(log n), and reading the smallest is O(1). Maintaining a fully sorted list may require O(n) insertion shifts.

## 18. BFS and shortest paths

Which graph traversal finds the fewest-edge route in an unweighted graph?

### Solution

BFS, using a FIFO queue. It explores vertices in increasing distance from the source.

## 19. Graph representation

Which representation is a good default for a sparse graph with millions of possible vertex pairs but relatively few edges?

### Solution

An adjacency list: O(V+E) storage versus O(V^2) for an adjacency matrix.

## 20. Dijkstra restriction

Why does Dijkstra require non-negative edge weights?

### Solution

Its greedy rule treats a smallest-known distance as final. A later negative edge could reduce that distance and invalidate the rule. Use an algorithm that supports negative edges and detects negative cycles.

## 21. Sliding-window precondition

Why does the shortest-subarray example shrink the left edge only when values are non-negative?

### Solution

With non-negative values, removing from the left cannot increase the sum. Negative values break that monotonic behavior, so the proof and algorithm no longer apply.

## 22. Algorithm review

Before optimizing a slow solution, what should you establish?

### Solution

Correct input/output contract, constraints, a correct simple baseline, a complexity estimate, and tested edge cases. Then identify the measured bottleneck and choose a better representation or algorithm.

## Challenge

Build a small route planner over a weighted graph. Requirements: return the shortest distance and route, reject negative weights, handle a missing route, and document graph representation, complexity, and tie behavior.
