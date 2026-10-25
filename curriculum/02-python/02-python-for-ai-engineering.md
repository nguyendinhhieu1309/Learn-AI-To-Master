# Python for AI Engineering

AI work needs more than notebook syntax. This lesson builds reliable habits for numerical code, reusable packages, and experiments.

## 1. Data structures and complexity intuition

Use lists for ordered mutable sequences, tuples for fixed records, dictionaries for keyed lookup, sets for membership and uniqueness, and generators for lazy iteration. Choosing a suitable data structure often matters more than a micro-optimization.

Dictionary and set membership are average-case constant time; scanning a list is linear in its length. Repeatedly checking membership in a list inside a large loop can turn an apparently simple task into quadratic work.

## 2. Functions, types, and contracts

Keep transformations small and explicit. A function should make its inputs, output, units, missing-value behavior, and side effects clear. Type hints improve editor support and make interfaces easier to review; they do not validate data at runtime.

```python
from collections.abc import Sequence

def normalize_texts(texts: Sequence[str]) -> list[str]:
    """Strip surrounding whitespace without mutating the input."""
    return [text.strip() for text in texts]
```

Raise a clear exception for invalid inputs instead of returning a plausible but incorrect default. Avoid broad `except Exception` blocks that hide bugs.

## 3. NumPy array thinking

Vectorize operations over arrays instead of writing Python loops for every element. Make dtype and shape deliberate:

```python
import numpy as np

features = np.asarray([[1.0, 2.0], [3.0, 4.0]], dtype=np.float32)
mean = features.mean(axis=0, keepdims=True)
scale = features.std(axis=0, keepdims=True).clip(min=1e-6)
standardized = (features - mean) / scale
assert standardized.shape == features.shape
```

`axis=0` computes per-column statistics here. `keepdims=True` preserves a broadcastable shape. Distinguish views from copies when slicing; mutating a view may modify the original array.

## 4. Notebooks versus modules

Use notebooks for exploration and explanation. Move stable transformations, metrics, and training code into importable modules so they can be reused and reviewed. Keep notebooks restartable from a clean kernel, avoid hidden state, and record inputs and output artifacts.

An example project layout:

```text
project/
  pyproject.toml
  README.md
  src/project_name/
    data.py
    features.py
    train.py
    evaluate.py
  configs/
  notebooks/
  tests/
```

## 5. Environments and dependencies

Use a virtual environment per project and pin direct dependencies in a lock file for reproducible deployments. Record Python and accelerator-library versions. Separate runtime dependencies from development tools. Do not commit credentials, local caches, datasets, or large model artifacts by accident.

## 6. Reproducible experiments

Pass configuration explicitly rather than relying on hidden globals. Set random seeds for Python, NumPy, and the ML framework, while recognizing GPU kernels may remain nondeterministic. Save configuration, data version, code revision, metrics, and checkpoint together.

## 7. Logging and error handling

Use structured logs with run IDs and stage names. Log counts, shapes, durations, and safe summaries; avoid dumping private records or full user prompts. Handle expected failures specifically, add context, and let unexpected exceptions surface with tracebacks.

## 8. Exercises

1. A loop checks whether each of a million IDs appears in a list of a million IDs. How could a set change the complexity?
2. Why does `axis=0` calculate per-feature means for a `(rows, features)` matrix?
3. What belongs in an experiment record to reproduce a model beyond just the random seed?
4. Why can notebook cells that only work after running earlier cells in a particular order be dangerous?
