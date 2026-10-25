# Python Basics for AI Learners

## Learning objectives

Read small Python programs, write functions, work with collections, and make errors easier to diagnose.

## 1. Values, names, and types

Python names refer to values. Common types include integers (`int`), decimals (`float`), text (`str`), booleans (`bool`), and missing value `None`. Use `=` to assign a name and `==` to compare values.

```python
temperature_c = 21.5
is_raining = False
city = "Hanoi"
```

## 2. Conditions and loops

Use `if/elif/else` to branch. Use `for` to process each item; use `while` when repetition depends on a changing condition. Prefer a `for` loop when iterating through a known collection.

```python
scores = [0.72, 0.91, 0.64]
for score in scores:
    if score >= 0.8:
        print("strong", score)
    else:
        print("review", score)
```

Indentation defines code blocks. A misaligned line is a syntax error or, worse, changes which lines belong to a condition.

## 3. Collections

- **List:** ordered and changeable, e.g. `["cat", "dog"]`.
- **Tuple:** ordered and commonly used for fixed groupings, e.g. `(height, width)`.
- **Dictionary:** key/value mapping, e.g. `{"name": "Mai", "score": 0.9}`.
- **Set:** unique values without a meaningful order.

Use a dictionary when a value has named fields; use a list when order is the main idea. Indexing starts at 0: `scores[0]` is the first score.

## 4. Functions make work reusable

```python
def mean(values):
    if len(values) == 0:
        raise ValueError("mean needs at least one value")
    return sum(values) / len(values)

print(mean([2, 4, 6]))  # 4.0
```

A function should have a clear job, meaningful parameter names, and an explicit result. If a function modifies a passed-in list or dictionary, say so; hidden mutation can make notebook behavior confusing.

## 5. Errors are information

Read the final line of a traceback first to find the error type and message, then inspect the line that failed. Common errors: `NameError` (unknown name), `TypeError` (operation on incompatible types), `KeyError` (missing dictionary key), `IndexError` (index out of range), and `ValueError` (value has the wrong form).

Use exceptions for exceptional input, not to hide every bug. Validate input near the boundary where it enters your program.

## 6. Read files safely

```python
from pathlib import Path

text = Path("notes.txt").read_text(encoding="utf-8")
print(text[:100])
```

Paths depend on the current working directory. When a file is not found, inspect the working directory and use an explicit project-relative path. For CSV data, use the `csv` module or pandas rather than splitting lines manually; quoted commas and missing fields make hand parsing fragile.

## Practice with answers

Write a function `count_above(values, threshold)` that counts numbers greater than the threshold. For `[1, 5, 5, 8]` with threshold 5, the answer is 1. Think about what should happen for an empty list: it should naturally return 0.

## Common mistakes

- Naming a variable `list`, `sum`, or `str`, hiding a built-in function.
- Mixing a number and a string, such as adding `3 + "4"` without conversion.
- Writing one huge notebook cell instead of small functions.
- Catching every exception and continuing with corrupted data.
