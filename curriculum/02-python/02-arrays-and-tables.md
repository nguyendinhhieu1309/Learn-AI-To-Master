# Arrays and Tables with NumPy and pandas

## Learning objectives

Understand the difference between a Python list, a NumPy array, and a pandas table; select data by position or name; and check shapes and missing values.

## 1. NumPy arrays

NumPy arrays store numerical data in a regular shape and support vectorized operations:

```python
import numpy as np

x = np.array([1.0, 2.0, 3.0])
weights = np.array([0.5, -1.0, 2.0])
score = x @ weights          # dot product: 4.5
scaled = 2 * x               # [2.0, 4.0, 6.0]
```

`x.shape` is `(3,)`. For a matrix, `X.shape` might be `(n_examples, n_features)`. Vectorized operations are concise and usually faster than Python loops, but they apply rules about broadcasting. If result shapes are surprising, print the shapes before proceeding.

## 2. pandas tables

A `DataFrame` is a table with labeled rows and columns. A `Series` is one labeled column.

```python
import pandas as pd

df = pd.DataFrame({
    "hours": [2.0, 4.0, 3.0],
    "passed": [False, True, True],
})
print(df.shape)          # (3, 2)
print(df["hours"].mean())
print(df.isna().sum())
```

Useful first checks include `df.head()`, `df.shape`, `df.dtypes`, `df.describe(include="all")`, `df.isna().sum()`, and `df.nunique()`. These checks reveal suspicious columns, missingness, and unexpected categories before modeling.

## 3. Select by label and condition

Use `df["column"]` to select a column. Use `.loc[rows, columns]` to select by labels and `.iloc[rows, columns]` by integer positions. Filter rows with a boolean condition:

```python
passed = df.loc[df["hours"] >= 3, ["hours", "passed"]]
```

When combining conditions, wrap each condition in parentheses and use `&` for AND, `|` for OR. Do not use Python's `and` with whole pandas columns.

## 4. Load data and mind types

```python
df = pd.read_csv("data/students.csv")
```

Inspect inferred types. A numeric column may be read as text if it contains values such as `"unknown"` or commas. Convert deliberately with `pd.to_numeric(..., errors="coerce")`, then inspect which values became missing rather than silently discarding them.

## Mini exercise

Create a table with three products, prices, and quantities. Add a `revenue` column (`price * quantity`), filter products with revenue above 100, and report total revenue. Check that the units are consistent (for example, dollars times units).

## Common mistakes

- Assuming array shape and axis meaning without checking them.
- Treating a missing value as zero without a domain reason.
- Chained assignment in pandas; create an explicit copy or use `.loc` to make the intended update clear.
- Running a calculation on a string column and trusting the result without inspecting types.
