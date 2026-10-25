# Functions, Modules, Files, and Errors

Functions divide a program into named pieces that can be reused, tested, and understood independently.

## Learning objectives

- Define functions with clear inputs, outputs, and side effects.
- Use positional, keyword, default, and variable-length arguments.
- Explain scope and write basic generators.
- Organize code into modules and handle files and errors safely.

## 1. Functions and return values

~~~python
def mean(values: list[float]) -> float:
    if not values:
        raise ValueError("mean requires at least one value")
    return sum(values) / len(values)

print(mean([2.0, 4.0, 6.0]))
~~~

Return sends a value to the caller. A function without a return statement produces None. Type hints describe intended types and help tools, but Python does not enforce them automatically.

Prefer one clear job per function. State units, empty-input behavior, mutation, and exceptions. Avoid functions that calculate, print, modify global state, and write files all at once.

## 2. Arguments and defaults

~~~python
def clip(value: float, low: float = 0.0, high: float = 1.0) -> float:
    return min(max(value, low), high)

clip(1.4)
clip(value=1.4, high=2.0)
~~~

Arguments may be positional or named. Use *args and **kwargs only when a flexible interface is genuinely useful.

Never use a mutable object as a default argument:

~~~python
def add_tag(tag: str, tags: list[str] | None = None) -> list[str]:
    if tags is None:
        tags = []
    tags.append(tag)
    return tags
~~~

Default expressions are evaluated once when the function is defined, so a list default can be shared across calls.

## 3. Scope, lambda, and closures

Python resolves names through local, enclosing, global, and built-in scopes. Pass dependencies as arguments rather than relying on mutable global variables.

Lambda is a small anonymous function. It is useful for simple keys:

~~~python
examples = [{"id": "b", "score": 0.7}, {"id": "a", "score": 0.9}]
ordered = sorted(examples, key=lambda item: item["score"], reverse=True)
~~~

Use a named function for multi-step logic. A closure remembers a name from the enclosing scope; closures created inside loops can surprise learners if they all read the same changing variable.

## 4. Iterables and generators

An iterable can produce an iterator. A generator yields items one at a time and can avoid storing a large intermediate list:

~~~python
def valid_rows(rows):
    for row in rows:
        if row.get("id") and row.get("label") is not None:
            yield row
~~~

A generator is usually consumed once. To make another pass, recreate it or store the values. Generators suit file streams and large datasets when a pipeline can process records incrementally.

## 5. Modules and packages

A module is a Python file that defines reusable code. Import what the program needs:

~~~python
from pathlib import Path
from collections import Counter
~~~

Separate reusable functions from command-line behavior:

~~~python
def main() -> None:
    print("run the program")

if __name__ == "__main__":
    main()
~~~

The guard allows a file to be run as a program or imported without automatically launching its workflow. Use a virtual environment for each project and record dependencies.

## 6. Files, paths, JSON, and CSV

Use Path objects and specify text encoding:

~~~python
from pathlib import Path

path = Path("data") / "records.txt"
text = path.read_text(encoding="utf-8")

with Path("report.txt").open("w", encoding="utf-8") as file:
    file.write("complete\n")
~~~

The with block closes a file even if an error occurs. Use the json module for JSON and csv.DictReader for CSV. Splitting CSV lines on commas fails on quoted fields containing commas or newlines.

## 7. Exceptions and validation

Catch a specific expected exception at the boundary where the program can respond:

~~~python
try:
    count = int(raw_count)
except ValueError as error:
    raise ValueError("count must be a whole number") from error
~~~

Do not catch every exception and continue with corrupted data. Raise an error with useful context for invalid input. Define a custom exception when it adds meaning to the public interface.

## Practice with answers

1. Why is return more reusable than print in a calculation function?
2. Why might a large-file function yield rows?
3. What bug can a mutable default argument cause?
4. Why use a context manager for files?

### Answers

1. The caller can inspect, combine, test, or save the value.
2. A generator processes one row at a time and reduces peak memory.
3. Calls can unexpectedly share one object.
4. It reliably closes the file, including after errors.
