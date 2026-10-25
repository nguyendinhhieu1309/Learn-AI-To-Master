# Hash Tables, Searching, and Sorting

This lesson explains how to find and order data, compare algorithm costs, and use hashing for fast keyed access.

## 1. Hash tables

A hash table maps a key to a location using a hash function. Different keys can map to the same location; this is a collision. Implementations must detect collisions and may resize when the table becomes crowded.

Python dictionaries and sets are hash-table-based. Lookup, insertion, and deletion are average-case O(1), but worst-case behavior can be O(n). Keys must have stable equality and hash behavior while stored. Strings, numbers, and tuples of hashable values are common keys.

~~~python
word_counts = {}
for word in ["ai", "data", "ai"]:
    word_counts[word] = word_counts.get(word, 0) + 1
~~~

Hash tables are not sorted maps. If ordered traversal matters, sort keys or use an appropriate ordered structure.

## 2. Linear search

Linear search checks values one by one:

~~~python
def linear_search(values: list[int], target: int) -> int:
    for index, value in enumerate(values):
        if value == target:
            return index
    return -1
~~~

Time is O(n), extra space is O(1). It works on unsorted data and may be the simplest option for small inputs.

## 3. Binary search

Binary search repeatedly halves a sorted search range. It takes O(log n) comparisons and O(1) extra space in an iterative implementation. The input must be sorted using the same ordering as the comparisons.

~~~python
def binary_search(values: list[int], target: int) -> int:
    left, right = 0, len(values) - 1
    while left <= right:
        middle = left + (right - left) // 2
        if values[middle] == target:
            return middle
        if values[middle] < target:
            left = middle + 1
        else:
            right = middle - 1
    return -1
~~~

The return value is one matching index, or -1 if absent. To find the first duplicate, change the search to continue left after a match. A common bug is using an inclusive range inconsistently, causing an infinite loop or skipped value.

## 4. Sorting algorithms

| Algorithm | Best time | Average time | Worst time | Extra space | Stable? |
| --- | --- | --- | --- | --- | --- |
| Bubble sort | O(n) with early stop | O(n^2) | O(n^2) | O(1) | Usually |
| Selection sort | O(n^2) | O(n^2) | O(n^2) | O(1) | Usually no |
| Insertion sort | O(n) | O(n^2) | O(n^2) | O(1) | Yes |
| Merge sort | O(n log n) | O(n log n) | O(n log n) | O(n) | Yes |
| Quick sort | O(n log n) | O(n log n) | O(n^2) | O(log n) average stack | Usually no |

“Stable” means equal-key items keep their relative input order. Stability matters when sorting records by multiple keys in stages.

~~~python
records = [{"name": "A", "score": 8}, {"name": "B", "score": 5}]
by_score = sorted(records, key=lambda row: row["score"])
~~~

Python's built-in sort is stable and highly optimized. Use it for application code; implement classic sorts to learn their behavior and complexity.

## 5. Choosing a search/sort strategy

- Unsorted small sequence: linear search.
- Many lookups on static data: sort once, then binary search, or build a set/dictionary if exact-key lookup is intended.
- Need ordered output: sort using a clear key.
- Need repeated minimum/maximum extraction: use a heap instead of sorting after every update.

Include preprocessing cost in comparisons. Sorting n items costs O(n log n); it is worthwhile for repeated searches only when the total workload justifies it.

## Practice with answers

1. What condition does binary search require?
2. What is the worst-case time of quick sort?
3. What does stable sorting preserve?
4. Why can a dictionary lookup not replace every binary search?

### Answers

1. The searched sequence must be sorted under the same ordering.
2. O(n^2), for unfavorable pivots.
3. Relative order among items with equal keys.
4. Binary search works by order and can answer range questions; dictionary lookup is for exact keys and does not preserve sorted search semantics.
