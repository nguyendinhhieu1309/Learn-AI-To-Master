# Recursion and Algorithm Complexity

An algorithm is a finite, unambiguous procedure for solving a problem. Complexity describes how its resource use grows as input grows.

## Learning objectives

- Estimate time and extra-space complexity.
- Read common Big-O classes and compare growth rates.
- Analyze loops, nested loops, and divide-and-conquer work.
- Write recursive functions with a correct base case.

## 1. What Big-O measures

Let n represent input size. Big-O describes an asymptotic upper bound on growth, ignoring machine-specific seconds and constant factors. It is not a benchmark: two O(n) programs can have very different constant costs.

Common growth classes, from slower growth to faster growth:

| Complexity | Common example | Growth when n doubles |
| --- | --- | --- |
| O(1) | list index or dictionary lookup on average | about the same |
| O(log n) | binary search | adds about one level |
| O(n) | scan every element | doubles |
| O(n log n) | comparison sort with good guarantees | a little more than doubles |
| O(n^2) | compare every pair | about four times |
| O(2^n) | enumerate every subset | roughly squares |
| O(n!) | enumerate every ordering | grows extremely quickly |

For large n, lower growth usually matters more than small constant improvements.

## 2. Count work and simplify

~~~python
def find_value(values, target):
    for value in values:
        if value == target:
            return True
    return False
~~~

The loop may inspect every item: O(n) time. It uses O(1) extra space.

~~~python
def print_all_pairs(values):
    for left in values:
        for right in values:
            print(left, right)
~~~

The nested loops do n times n work: O(n^2). If loops process separate input sizes n and m, complexity may be O(nm), not O(n^2).

Ignore constants and lower-order terms for asymptotic class: 5n+20 is O(n), and n^2+3n is O(n^2). State whether a result is worst-case, average-case, or best-case when it matters.

## 3. Time and space complexity

Time complexity estimates operation growth. Space complexity estimates memory growth. Include auxiliary storage and recursion stack, but distinguish it from the input storage already provided.

An algorithm that copies an n-item list uses O(n) extra space. A loop with a few counters uses O(1) extra space. A recursive call chain of depth n uses O(n) call-stack space.

## 4. Amortized cost

Appending to a Python list is amortized O(1): most appends are constant work, while occasional capacity growth copies elements. The average over a long sequence remains constant per append. This differs from claiming every individual append costs exactly O(1).

## 5. Recursion

A recursive function calls itself on a smaller problem. It needs:

1. a base case that stops recursion;
2. progress toward that base case;
3. a result-combination rule.

~~~python
def factorial(n: int) -> int:
    if n < 0:
        raise ValueError("n must be non-negative")
    if n <= 1:
        return 1
    return n * factorial(n - 1)
~~~

This takes O(n) time and O(n) call-stack space. Python limits recursion depth; for large linear-depth problems use an iterative approach or an explicit stack.

## 6. Divide and conquer

Binary search discards half the remaining sorted range per step, giving O(log n) comparisons. Merge sort splits the input, sorts both halves, and merges them: the recurrence is T(n)=2T(n/2)+O(n), which yields O(n log n) time.

The “halve the input” pattern often suggests logarithmic recursion, but account for work done at every level and the cost of combining results.

## Practice with answers

1. What is the complexity of scanning one list and then another list of lengths n and m?
2. What is the complexity of two nested loops over the same n items?
3. What is the auxiliary stack space for factorial(n)?
4. Why is list append called amortized O(1)?

### Answers

1. O(n+m).
2. O(n^2).
3. O(n) recursive frames.
4. Most appends are constant work, with occasional expensive resizing spread across many operations.
