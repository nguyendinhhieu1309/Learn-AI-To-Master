# Algorithmic Problem-Solving Patterns

This lesson turns a problem statement into a clear algorithm. Patterns are reusable starting points, not rules to force onto every problem.

## 1. A repeatable workflow

1. Restate the task and define input/output precisely.
2. Work a tiny example by hand, including empty input and duplicates.
3. Identify constraints: input size, sortedness, memory, and whether updates occur.
4. Write the simplest correct algorithm first.
5. Estimate time and extra space.
6. Check edge cases and prove the key invariant.
7. Implement, test, and compare with a brute-force version on small inputs.

“Hard” is not a complexity class. Difficulty often comes from recognizing a useful representation or proving correctness; Big-O describes growth after an algorithm is chosen.

## 2. Frequency map

Count occurrences with a dictionary in O(n) expected time:

~~~python
def frequencies(values):
    count = {}
    for value in values:
        count[value] = count.get(value, 0) + 1
    return count
~~~

Use this for duplicate detection, anagrams, category counts, and matching by key.

## 3. Two pointers

For a sorted list, pointers can move inward without checking every pair:

~~~python
def has_pair_sum(sorted_values: list[int], target: int) -> bool:
    left, right = 0, len(sorted_values) - 1
    while left < right:
        total = sorted_values[left] + sorted_values[right]
        if total == target:
            return True
        if total < target:
            left += 1
        else:
            right -= 1
    return False
~~~

Time is O(n), extra space O(1). Sorting first costs O(n log n); do not forget that cost if the original input is unsorted.

## 4. Sliding window

For a contiguous segment, keep a window and update its summary as the edges move. For non-negative values, the following finds the shortest segment whose sum reaches a target:

~~~python
def shortest_at_least(target: int, values: list[int]) -> int:
    left = 0
    total = 0
    best = len(values) + 1
    for right, value in enumerate(values):
        total += value
        while total >= target:
            best = min(best, right - left + 1)
            total -= values[left]
            left += 1
    return 0 if best > len(values) else best
~~~

This window logic depends on non-negative values. With negatives, expanding or shrinking no longer changes the sum monotonically; a different method is needed.

## 5. Prefix sums

Define prefix[i] as the sum of the first i values. Then sum of values from index left through right inclusive is prefix[right+1]-prefix[left].

~~~python
def prefix_sums(values: list[int]) -> list[int]:
    prefix = [0]
    for value in values:
        prefix.append(prefix[-1] + value)
    return prefix
~~~

Build in O(n), answer each range-sum query in O(1), using O(n) memory.

## 6. Binary search on a condition

Binary search can find the first position where a monotone condition changes from false to true. State the invariant and update bounds carefully. The condition must remain monotone across the search range.

Examples include first value >= target in a sorted array or smallest capacity that can complete a set of jobs. This is different from guessing a midpoint without a proof that one half can be discarded.

## 7. Stack and monotonic stack

A stack is useful for nested structure and “next greater/smaller” questions. Maintain values in increasing or decreasing order; remove elements that can no longer be an answer. Each item is pushed and popped at most once, so the whole algorithm is often O(n), despite a while loop inside a for loop.

Use this pattern for matching parentheses, next greater element, and histogram rectangle problems.

## 8. Graph search and dynamic programming

Use BFS for shortest paths by number of edges in an unweighted graph. Use DFS for reachability, components, and backtracking. Use dynamic programming when a problem has overlapping subproblems and an optimal substructure; define the state and recurrence before coding.

## Practice with answer checks

1. Why does a two-pointer pair-sum method require sorted input?
2. Why does the sliding-window example fail with arbitrary negative numbers?
3. How can a nested while loop still be O(n) overall in a monotonic-stack algorithm?
4. What should be written before implementing dynamic programming?

### Answers

1. The sum direction tells which pointer can safely move.
2. Removing a value may increase the total, so the invariant no longer holds.
3. Each item is pushed once and removed once, so total pointer/stack operations are linear.
4. State meaning, base cases, recurrence, dependency order, and answer extraction.
