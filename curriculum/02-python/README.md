# Python and Computing Foundations

The code examples in this module use Python 3.10 or newer.

## Outcomes

- Write small, readable Python programs and reusable functions.
- Understand object-oriented programming and choose built-in structures for the task.
- Analyze time and space complexity and implement core data structures and algorithms.
- Work with files, tables, notebooks, packages, and version control.
- Debug common errors and make a result reproducible.

## Suggested sequence

1. [Python basics for AI learners](01-python-basics.md)
2. [Core language: types, strings, and collections](04-python-core-language.md)
3. [Functions, modules, files, and errors](05-functions-modules-files-and-errors.md)
4. [Arrays and tables with NumPy and pandas](02-arrays-and-tables.md)
5. [Object-oriented programming](06-object-oriented-programming.md)
6. [Recursion and algorithm complexity](07-recursion-and-complexity.md)
7. [Built-in data structures and their costs](08-built-in-data-structures.md)
8. [Linked lists, stacks, and queues](09-linked-lists-stacks-and-queues.md)
9. [Hash tables, searching, and sorting](10-hash-tables-searching-and-sorting.md)
10. [Trees, binary search trees, and heaps](11-trees-bst-and-heaps.md)
11. [Graphs, traversal, and shortest paths](12-graphs-and-graph-algorithms.md)
12. [Problem-solving patterns](13-algorithmic-problem-solving-patterns.md)
13. [Reproducible Python lab](03-reproducible-python-lab.md)
14. [Python and DSA problem set with solutions](PROBLEM_SET_WITH_SOLUTIONS.md)
15. Then study Git, environments, and software engineering in [AI Engineering](../11-ai-engineering/README.md).

## Practice

- Write a function that summarizes a list of numbers and handles an empty list.
- Implement linked list, BST, and graph operations after analyzing their complexity.
- Explain how binary search differs from linear search and when binary search is valid.
- Read a text or CSV file, validate its rows, and report malformed input clearly.
- Refactor a notebook cell sequence into reusable functions.
- Use Git commits to record a small change and inspect the diff.
- Complete the [CSV audit lab](03-reproducible-python-lab.md), including its failure-case tests.

## Checkpoint project

Create a command-line or notebook program that loads a small public dataset, reports its dimensions and basic summary statistics, handles a missing value, and saves a cleaned copy. Document how to reproduce it.

## Common mistakes to watch for

- Using notebooks as a substitute for organizing reusable code.
- Installing packages globally and losing track of the environment.
- Ignoring shapes, types, and missing values until model training fails.
- Making large commits that hide what changed.
- Using a linked list or handwritten sort by default when Python's list, dict, set, deque, or built-in sort is a better fit.
