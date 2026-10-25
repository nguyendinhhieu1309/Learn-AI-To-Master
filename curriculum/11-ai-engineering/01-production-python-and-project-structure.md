# Production Python and Project Structure

## Learning objectives

Organize an AI project so a teammate can install it, find its responsibilities, and run the same workflow without relying on notebook state.

## 1. From notebook to project

A notebook is useful for exploration. Production logic belongs in importable functions/modules so it can be reused by a command, API, scheduled job, or test. Keep notebooks for experiments and reports; move stable functions into a package.

```text
my-ai-project/
├── pyproject.toml       # package and tool configuration
├── README.md            # setup and usage
├── src/my_ai_project/
│   ├── data.py
│   ├── features.py
│   ├── model.py
│   └── inference.py
├── tests/
└── notebooks/
```

Separate responsibilities: data reading should not secretly train a model; model prediction should not depend on a cell having been run earlier.

## 2. Environments and dependencies

Use a project environment (`venv`, Conda, or another manager) so dependencies do not conflict across projects. Record direct dependencies and lock resolved versions for deployed or reproducibility-sensitive work. Pinning everything forever can block security fixes; update deliberately and rerun the relevant checks.

Keep configuration outside code: file paths, model names, batch size, thresholds, and service endpoints should be settings. Keep credentials in a secret manager or environment injection, never in source control, notebooks, logs, or a skill file.

## 3. Functions, types, and errors

Prefer small functions with clear inputs and outputs. Type hints document expected structure and enable static tools; they do not replace runtime validation on untrusted inputs.

```python
from pathlib import Path

def load_text(path: Path, *, encoding: str = "utf-8") -> str:
    """Read a UTF-8 text file or raise a useful file/encoding error."""
    return path.read_text(encoding=encoding)
```

Raise a specific error when the caller can respond. Do not convert every error into `None`, an empty dataset, or a confident model answer. Include enough context to debug while avoiding sensitive values.

## 4. Logging and configuration

Use structured logs with timestamps, operation names, model/data versions, request IDs, and durations. Keep metrics separate from verbose logs. Never log passwords, API keys, raw personal data, or full prompts by default. Read settings once at the application boundary and pass required values into functions explicitly.

## 5. Reproducibility

Record code revision, data version, model version, parameters, environment, split definition, and metrics. A random seed makes pseudorandom choices reproducible only within limits; GPU kernels, package versions, and concurrent execution can still vary.

## Practice

Extract a data-cleaning step and prediction step from a notebook into functions. Run them from a fresh environment. Change the input path through configuration, provoke a missing-file error, and check that the failure message explains the problem without exposing private data.

## Common mistakes

- Relying on notebook execution order or hidden global variables.
- Putting model, API, data, and UI logic in one large file.
- Installing dependencies without recording how the environment was created.
- Storing secrets in `.env` files that are accidentally committed or in notebook outputs.
