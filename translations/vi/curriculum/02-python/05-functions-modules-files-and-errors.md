# Function, module, file và lỗi

[Xem bài gốc bằng tiếng Anh](../../../../curriculum/02-python/05-functions-modules-files-and-errors.md)

Function chia chương trình thành các phần có tên để có thể tái sử dụng, kiểm tra và hiểu độc lập.

## Mục tiêu học tập

- Định nghĩa function với input, output và side effect rõ ràng.
- Dùng positional, keyword, default và variable-length argument.
- Giải thích scope và viết generator cơ bản.
- Tổ chức code thành module và xử lý file, lỗi an toàn.

## 1. Function và giá trị trả về

~~~python
def mean(values: list[float]) -> float:
    if not values:
        raise ValueError("mean requires at least one value")
    return sum(values) / len(values)

print(mean([2.0, 4.0, 6.0]))
~~~

return gửi giá trị về cho caller. Function không có câu lệnh return sẽ tạo ra None. Type hint mô tả kiểu dự kiến và hỗ trợ công cụ, nhưng Python không tự cưỡng chế các kiểu đó.

Ưu tiên mỗi function làm một việc rõ ràng. Nêu đơn vị, cách xử lý input rỗng, mutation và exception. Tránh function vừa tính toán, in ra, sửa global state, vừa ghi file.

## 2. Argument và giá trị mặc định

~~~python
def clip(value: float, low: float = 0.0, high: float = 1.0) -> float:
    return min(max(value, low), high)

clip(1.4)
clip(value=1.4, high=2.0)
~~~

Argument có thể truyền theo vị trí hoặc theo tên. Chỉ dùng *args và **kwargs khi interface linh hoạt thực sự cần thiết.

Không dùng object có thể thay đổi làm default argument:

~~~python
def add_tag(tag: str, tags: list[str] | None = None) -> list[str]:
    if tags is None:
        tags = []
    tags.append(tag)
    return tags
~~~

Biểu thức mặc định được đánh giá một lần khi function được định nghĩa, vì vậy list làm default có thể bị dùng chung giữa nhiều lần gọi.

## 3. Scope, lambda và closure

Python tìm tên qua scope local, enclosing, global và built-in. Truyền dependency qua argument thay vì dựa vào global variable có thể thay đổi.

Lambda là function ẩn danh ngắn. Nó hữu ích cho key đơn giản:

~~~python
examples = [{"id": "b", "score": 0.7}, {"id": "a", "score": 0.9}]
ordered = sorted(examples, key=lambda item: item["score"], reverse=True)
~~~

Dùng function có tên cho logic nhiều bước. Closure ghi nhớ tên trong enclosing scope; closure tạo bên trong loop có thể gây bất ngờ nếu tất cả cùng đọc một biến đang thay đổi.

## 4. Iterable và generator

Iterable có thể tạo ra iterator. Generator yield từng phần tử một, tránh phải giữ list trung gian lớn trong bộ nhớ:

~~~python
def valid_rows(rows):
    for row in rows:
        if row.get("id") and row.get("label") is not None:
            yield row
~~~

Generator thường chỉ được tiêu thụ một lần. Muốn duyệt lại, hãy tạo generator mới hoặc lưu các giá trị. Generator phù hợp với luồng file và dataset lớn khi pipeline có thể xử lý từng record.

## 5. Module và package

Module là file Python định nghĩa code có thể tái sử dụng. Import những thành phần chương trình cần dùng:

~~~python
from pathlib import Path
from collections import Counter
~~~

Tách function tái sử dụng khỏi hành vi command line:

~~~python
def main() -> None:
    print("run the program")

if __name__ == "__main__":
    main()
~~~

Guard này cho phép chạy file như một chương trình hoặc import file mà không tự động khởi chạy workflow. Dùng virtual environment riêng cho mỗi project và ghi lại dependency.

## 6. File, path, JSON và CSV

Dùng object Path và chỉ định text encoding:

~~~python
from pathlib import Path

path = Path("data") / "records.txt"
text = path.read_text(encoding="utf-8")

with Path("report.txt").open("w", encoding="utf-8") as file:
    file.write("complete\n")
~~~

Khối with đóng file ngay cả khi có lỗi. Dùng module json cho JSON và csv.DictReader cho CSV. Tự tách dòng CSV theo dấu phẩy sẽ sai với field đặt trong ngoặc kép có dấu phẩy hoặc xuống dòng.

## 7. Exception và validation

Bắt exception cụ thể có thể dự đoán ở ranh giới nơi chương trình có thể xử lý:

~~~python
try:
    count = int(raw_count)
except ValueError as error:
    raise ValueError("count must be a whole number") from error
~~~

Không bắt mọi exception rồi tiếp tục với dữ liệu đã hỏng. Với input sai, hãy raise lỗi có ngữ cảnh hữu ích. Định nghĩa custom exception khi nó giúp interface công khai dễ hiểu hơn.

## Bài luyện tập kèm đáp án

1. Vì sao return tái sử dụng tốt hơn print trong function tính toán?
2. Vì sao function xử lý file lớn có thể yield từng hàng?
3. Mutable default argument có thể gây bug gì?
4. Vì sao dùng context manager cho file?

### Đáp án

1. Caller có thể kiểm tra, kết hợp, test hoặc lưu giá trị.
2. Generator xử lý từng hàng và giảm bộ nhớ cực đại.
3. Nhiều lần gọi có thể bất ngờ dùng chung một object.
4. Nó đóng file đáng tin cậy, kể cả khi xảy ra lỗi.
