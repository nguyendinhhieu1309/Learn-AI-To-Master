# Python Core Language

This lesson builds a mental model for values, names, control flow, and the built-in collections used in data science and AI.

## Learning objectives

- Write expressions, branches, and loops.
- Choose among strings, lists, tuples, sets, and dictionaries.
- Explain indexing, slicing, mutability, and references.
- Use comprehensions without hiding complicated logic.

## 1. Values and types

A variable name refers to an object. Assignment binds a name; it does not necessarily copy the object.

~~~python
age = 21
temperature = 22.5
name = "Linh"
is_ready = True
result = None
~~~

Common types include integers, floating-point numbers, strings, booleans, and None. Python is dynamically typed: a name can be rebound, but operations still have type rules. Convert at input boundaries with int, float, or str; invalid conversions raise ValueError.

## 2. Operators and comparisons

Arithmetic includes addition, subtraction, multiplication, division, floor division, remainder, and exponentiation. Comparisons return booleans. Use in for membership, == for value equality, and is for object identity.

Use is None to test for None:

~~~python
if label is None:
    print("label is missing")
~~~

Do not use is to compare ordinary strings or numbers. Combine conditions with and, or, and not. Parenthesize mixed comparisons for readability.

## 3. Strings

Strings are immutable sequences of Unicode characters. Indexing starts at zero; negative indexes count from the end. Slices include the start and exclude the stop.

~~~python
word = "robotics"
print(word[0])      # r
print(word[-1])     # s
print(word[0:5])    # robot
print(word[::2])    # every second character
~~~

Useful methods include strip, lower, split, replace, and startswith. Use f-strings for formatted output:

~~~python
model_name = "baseline"
accuracy = 0.875
print(f"{model_name}: {accuracy:.1%}")
~~~

## 4. Lists and tuples

A list is ordered and mutable. A tuple is ordered and immutable after creation. Lists support append, extend, insert, remove, pop, sort, and copy.

~~~python
features = [0.2, 1.4, 3.0]
features.append(4.1)
first_two = features[:2]
coordinates = (12.5, 8.0)
x, y = coordinates
~~~

sorted(values) returns a new sorted list; values.sort() changes the existing list. Avoid removing from a list while iterating over it because elements shift.

## 5. Dictionaries and sets

A dictionary maps hashable keys to values. A set stores unique hashable values. Both are useful for repeated membership checks and keyed lookup.

~~~python
scores = {"image_a": 0.92, "image_b": 0.73}
scores["image_c"] = 0.88
print(scores.get("unknown", 0.0))

seen_ids = {"a17", "b29"}
print("a17" in seen_ids)
~~~

Use get when a missing key has a deliberate default. Lists and dictionaries cannot be keys because they are mutable and unhashable.

## 6. Branches, loops, and comprehensions

~~~python
for score in [0.72, 0.91, 0.64]:
    if score >= 0.8:
        print("passed")
    else:
        print("review")

positive = [value for value in values if value > 0]
~~~

Use for to process a collection or known range. Use while when repetition depends on a changing condition. Ensure that every while loop can reach its stopping condition. Use a regular loop when a comprehension would contain several conditions or side effects.

## 7. Mutability, aliasing, and copying

Two names can refer to the same mutable list:

~~~python
original = [1, 2]
alias = original
alias.append(3)
print(original)  # [1, 2, 3]
~~~

list.copy creates a shallow copy; nested objects remain shared. Deep copying is more expensive and should be used only when an independent recursive copy is required.

## Practice with answer checks

1. Difference between value equality and identity?
2. Why does changing a list through an alias affect the original name?
3. Which structure naturally represents unique labels?
4. What does [2, 5, 8, 11][1:3] return?

### Answers

1. Equality compares values; identity checks whether names refer to the same object.
2. Both names refer to the same mutable list.
3. A set.
4. [5, 8]; the stop index is excluded.
