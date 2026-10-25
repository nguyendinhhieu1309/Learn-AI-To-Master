# Thực hành Python tái lập: từ CSV đến chương trình có kiểm thử

[Xem bài gốc bằng tiếng Anh](../../../../curriculum/02-python/03-reproducible-python-lab.md)

Lab này thực hành các thói quen Python giúp người khác hiểu được thí nghiệm AI: tách input/output khỏi phép tính, kiểm tra dữ liệu ở ranh giới, biến lỗi thành thông tin hữu ích và kiểm tra cả trường hợp biên.

## Mục tiêu học tập

- Chuyển tác vụ notebook dùng một lần thành chương trình nhỏ có function tái sử dụng.
- Phân tích CSV mà không âm thầm chấp nhận dữ liệu sai định dạng.
- Kiểm tra trường hợp bình thường và trường hợp lỗi chỉ với standard library của Python.
- Nêu rõ lệnh chạy và quy ước đầu ra.

## Điều kiện tiên quyết

Hoàn thành [Python cơ bản](01-python-basics.md) và [array, bảng dữ liệu](02-arrays-and-tables.md), hoặc đã biết function, dictionary, vòng lặp, path và CSV.

Ví dụ dùng Python 3.10 trở lên.

## 1. Xác định contract trước

Chương trình nhận CSV có các cột record_id, age và label. Báo cáo gồm:

- số hàng dữ liệu;
- số giá trị thiếu theo từng cột bắt buộc;
- số tuổi không hợp lệ;
- số nhãn nằm ngoài tập cho phép $\{0,1\}$.

Chương trình không tự đoán cách sửa record. Báo lỗi an toàn hơn việc âm thầm đổi ý nghĩa dữ liệu.

## 2. Tách parsing khỏi báo cáo

Tạo file audit.py:

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

Ranh giới file chịu trách nhiệm đọc dữ liệu và kiểm tra schema. audit_rows nhận record đã parse nên có thể kiểm tra mà không cần tạo file. Số hàng bắt đầu từ 2 vì dòng 1 là header.

## 3. Viết test cho input thông thường và bất thường

Tạo file test_audit.py cùng thư mục với audit.py:

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

Chạy lab từ thư mục chứa cả hai file:

~~~text
python -m unittest -v
python audit.py data.csv
~~~

Trường bắt buộc rỗng được tính là thiếu. Giá trị không rỗng nhưng sai định dạng được báo riêng. Một hàng có thể mắc cả hai loại lỗi; báo cáo chủ ý giữ lại cả hai thông tin.

## 4. Mở rộng contract mà không che giả định

Mỗi lần chỉ thêm một cải tiến:

1. Trả về kiểu report có cấu trúc thay vì dictionary tổng quát.
2. Thêm khoảng tuổi cấu hình được chỉ khi domain dữ liệu có cơ sở.
3. Thêm tên input và content hash vào metadata của báo cáo.
4. Ghi JSON ra file tạm rồi chỉ đổi tên sau khi chạy thành công.
5. Thêm test cho file rỗng và CSV có dấu phẩy nằm trong dấu ngoặc kép.

Không đổi “không hợp lệ” thành “thiếu” nếu không ghi nhận lý do. Không để audit âm thầm loại hàng.

## 5. Checklist gỡ lỗi

- Kiểm tra shape input và tên cột trước khi đổi giá trị.
- Xem một hàng tiêu biểu sau khi CSV reader parse.
- Kiểm tra type ở ranh giới; giá trị CSV ban đầu đều là chuỗi.
- Phân biệt exception do thiếu input với lỗi nằm trong một hàng.
- Lưu ví dụ nhỏ gây lỗi thành regression test.

## Bài luyện tập

1. Thêm argument tuổi tối đa và báo tuổi vượt giới hạn đã cấu hình.
2. Kiểm tra record_id có trùng không. Trả về ID trùng và số hàng tương ứng.
3. Thêm tùy chọn xuất JSON để tạo báo cáo máy có thể đọc.
4. Giải thích hành vi nào nên nằm trong audit_rows, hành vi nào thuộc main.

## Gợi ý đáp án

1. Chỉ parse tuổi sau khi kiểm tra giá trị không rỗng và là số. Dùng list riêng cho các hàng ngoài khoảng; không ghi đè giá trị gốc.
2. Duy trì map từ ID sang số hàng xuất hiện đầu tiên. Khi gặp lại, ghi nhận cả hai số hàng. Quyết định rõ cách xử lý ID rỗng.
3. Serialize report bằng JSON encoder chuẩn. Chuyển path và object tùy chỉnh thành chuỗi hoặc kiểu nguyên thủy trước.
4. audit_rows chứa quy tắc dữ liệu deterministic và dễ kiểm tra. main xử lý command-line argument, file và cách trình bày.

## Lỗi thường gặp

- Bắt mọi exception rồi in “đã xảy ra lỗi”, làm che mất bug lập trình.
- Sửa input record trong lúc báo cáo chất lượng.
- Chỉ test dataset sạch.
- Hard-code ngưỡng domain mà không ghi rõ nguồn gốc.
- Cho rằng parse thành công đồng nghĩa dữ liệu hợp lệ.
