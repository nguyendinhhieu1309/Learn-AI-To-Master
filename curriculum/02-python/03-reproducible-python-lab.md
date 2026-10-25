# Reproducible Python Lab: From CSV to Tested Program

This lab practices the Python habits that make AI experiments understandable to another person: separate input/output from calculations, validate data at the boundary, make errors actionable, and test edge cases.

## Learning objectives

- Turn a one-off notebook task into a small program with reusable functions.
- Parse a CSV without silently accepting malformed data.
- Test normal cases and failure cases using only the Python standard library.
- Make the run command and output contract explicit.

## Prerequisites

Complete [Python basics](01-python-basics.md) and [arrays and tables](02-arrays-and-tables.md), or know functions, dictionaries, loops, paths, and CSV files.

The examples use Python 3.10 or newer.

## 1. Define the contract first

The program accepts a CSV with columns `record_id`, `age`, and `label`. It reports:

- number of data rows;
- number of missing values per required column;
- number of invalid ages;
- number of labels outside the allowed set $\{0,1\}$.

It does not guess how to repair records. Reporting a defect is safer than silently changing the meaning of the data.

## 2. Keep parsing separate from reporting

Create `audit.py`:

~~~python
from __future__ import annotations

import argparse
import csv
import math
from collections import Counter
from pathlib import Path
from typing import Any

REQUIRED_COLUMNS = ("record_id", "age", "label")
ALLOWED_LABELS = {"0", "1"}


def audit_rows(rows: list[dict[str, str | None]]) -> dict[str, Any]:
    missing = Counter()
    invalid_age_rows: list[int] = []
    invalid_label_rows: list[int] = []

    for row_number, row in enumerate(rows, start=2):
        for column in REQUIRED_COLUMNS:
            if not (row.get(column) or "").strip():
                missing[column] += 1

        raw_age = (row.get("age") or "").strip()
        if raw_age:
            try:
                age = float(raw_age)
            except ValueError:
                invalid_age_rows.append(row_number)
            else:
                if not math.isfinite(age) or age < 0:
                    invalid_age_rows.append(row_number)

        raw_label = (row.get("label") or "").strip()
        if raw_label and raw_label not in ALLOWED_LABELS:
            invalid_label_rows.append(row_number)

    return {
        "row_count": len(rows),
        "missing_by_column": dict(missing),
        "invalid_age_rows": invalid_age_rows,
        "invalid_label_rows": invalid_label_rows,
    }


def audit_csv(path: Path) -> dict[str, Any]:
    with path.open("r", encoding="utf-8", newline="") as file:
        reader = csv.DictReader(file)
        if reader.fieldnames is None:
            raise ValueError("CSV is empty or has no header row.")

        missing_columns = set(REQUIRED_COLUMNS) - set(reader.fieldnames)
        if missing_columns:
            names = ", ".join(sorted(missing_columns))
            raise ValueError(f"Missing required columns: {names}")

        return audit_rows(list(reader))


def main() -> None:
    parser = argparse.ArgumentParser(description="Audit a small CSV dataset.")
    parser.add_argument("csv_path", type=Path, help="Path to the input CSV")
    args = parser.parse_args()

    report = audit_csv(args.csv_path)
    print(f"Rows: {report['row_count']}")
    print(f"Missing: {report['missing_by_column']}")
    print(f"Invalid age rows: {report['invalid_age_rows']}")
    print(f"Invalid label rows: {report['invalid_label_rows']}")


if __name__ == "__main__":
    main()
~~~

The file boundary handles reading and schema checks. `audit_rows` receives already parsed records, so it can be tested without creating a file. The row number starts at 2 because line 1 contains the header.

## 3. Add tests for ordinary and adversarial inputs

Create `test_audit.py` beside `audit.py`:

~~~python
import tempfile
import unittest
from pathlib import Path

from audit import audit_csv, audit_rows


class AuditRowsTests(unittest.TestCase):
    def test_counts_missing_and_invalid_values(self) -> None:
        rows = [
            {"record_id": "a", "age": "20", "label": "1"},
            {"record_id": "", "age": "unknown", "label": "3"},
            {"record_id": "c", "age": "-2", "label": None},
        ]

        report = audit_rows(rows)

        self.assertEqual(report["row_count"], 3)
        self.assertEqual(report["missing_by_column"]["record_id"], 1)
        self.assertEqual(report["missing_by_column"]["label"], 1)
        self.assertEqual(report["invalid_age_rows"], [3, 4])
        self.assertEqual(report["invalid_label_rows"], [3])

    def test_rejects_missing_required_column(self) -> None:
        with tempfile.TemporaryDirectory() as directory:
            path = Path(directory) / "small.csv"
            path.write_text("record_id,age\na,20\n", encoding="utf-8")

            with self.assertRaisesRegex(ValueError, "label"):
                audit_csv(path)


if __name__ == "__main__":
    unittest.main()
~~~

Run the lab from the directory containing both files:

~~~text
python -m unittest -v
python audit.py data.csv
~~~

An empty required field is counted as missing. An invalid non-empty value is reported separately. A row can have both kinds of defect; the report intentionally preserves both facts.

## 4. Extend the contract without hiding assumptions

Add one improvement at a time:

1. Return a structured report type instead of a generic dictionary.
2. Add a configurable age range only when the data domain justifies it.
3. Include the input filename and a content hash in report metadata.
4. Write JSON output to a temporary file and rename it only after a successful run.
5. Add a test for an empty file and for a CSV with a quoted comma.

Do not turn “invalid” into “missing” without recording the reason. Do not make the audit silently drop rows.

## 5. Debugging checklist

- Inspect input shape and column names before changing values.
- Inspect a representative row as parsed by the CSV reader.
- Check types at the boundary; CSV values begin as strings.
- Distinguish an exception caused by missing input from a defect in a row.
- Keep a small failing example as a regression test.

## Practice

1. Add a maximum-age argument and report ages above the configured bound.
2. Add a uniqueness check for `record_id`. Return duplicate identifiers and their row numbers.
3. Add a JSON output option that emits a machine-readable report.
4. Explain which behaviors belong in `audit_rows` and which belong in `main`.

## Answer guide

1. Parse the age only after checking it is non-empty and numeric. Keep a separate list for out-of-range rows; do not overwrite the original value.
2. Maintain a map from identifier to first row number. On a second occurrence, record both row numbers. Decide explicitly how blank identifiers behave.
3. Serialize the report with the standard JSON encoder. Convert paths and any custom objects into strings or primitive values first.
4. `audit_rows` contains deterministic data rules and is easy to test. `main` handles command-line arguments, files, and presentation.

## Common mistakes

- Catching every exception and printing “something went wrong,” which hides programming bugs.
- Mutating input records while reporting quality.
- Testing only a clean dataset.
- Hard-coding a domain threshold without documenting where it came from.
- Assuming that a successful parse means the data is valid.
