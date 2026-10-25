# Python cơ bản cho người học AI

[Xem bài gốc bằng tiếng Anh](../../../../curriculum/02-python/01-python-basics.md)

## Mục tiêu học tập

Đọc được chương trình Python ngắn, viết hàm, làm việc với collection và chẩn đoán lỗi dễ dàng hơn.

## 1. Giá trị, tên và kiểu dữ liệu

Tên trong Python tham chiếu đến các giá trị. Những kiểu thường gặp gồm số nguyên (int), số thực (float), văn bản (str), giá trị đúng/sai (bool) và giá trị vắng mặt None. Dùng dấu bằng để gán tên và toán tử so sánh bằng để so sánh giá trị.

~~~python
temperature_c = 21.5
is_raining = False
city = "Hanoi"
~~~

## 2. Điều kiện và vòng lặp

Dùng if/elif/else để rẽ nhánh. Dùng for để xử lý từng phần tử; dùng while khi số lần lặp phụ thuộc vào một điều kiện thay đổi. Nếu duyệt qua một collection đã biết, ưu tiên vòng lặp for.

~~~python
scores = [0.72, 0.91, 0.64]
for score in scores:
    if score >= 0.8:
        print("strong", score)
    else:
        print("review", score)
~~~

Indentation xác định khối code. Dòng thụt lề sai có thể gây syntax error, hoặc nghiêm trọng hơn là làm thay đổi những dòng thuộc về điều kiện.

## 3. Các collection

- **List:** có thứ tự và thay đổi được, ví dụ list gồm "cat" và "dog".
- **Tuple:** có thứ tự, thường dùng cho nhóm giá trị cố định, ví dụ cặp (height, width).
- **Dictionary:** ánh xạ key sang value, ví dụ mapping giữa name là Mai và score là 0.9.
- **Set:** chứa các giá trị duy nhất, không có thứ tự mang ý nghĩa.

Dùng dictionary khi các giá trị có tên trường; dùng list khi thứ tự là điều quan trọng. Index bắt đầu từ 0: phần tử scores[0] là điểm đầu tiên.

## 4. Dùng function để tái sử dụng công việc

~~~python
def mean(values):
    if len(values) == 0:
        raise ValueError("mean needs at least one value")
    return sum(values) / len(values)

print(mean([2, 4, 6]))  # 4.0
~~~

Một function nên có nhiệm vụ rõ ràng, tên parameter có ý nghĩa và kết quả tường minh. Nếu function sửa list hoặc dictionary được truyền vào, hãy nói rõ; thay đổi ngầm có thể khiến hành vi trong notebook khó hiểu.

## 5. Lỗi cung cấp thông tin

Đọc dòng cuối của traceback trước để tìm loại lỗi và thông báo, sau đó xem dòng code gây lỗi. Các lỗi thường gặp: NameError (tên chưa được định nghĩa), TypeError (thao tác trên kiểu không tương thích), KeyError (thiếu key trong dictionary), IndexError (index ngoài phạm vi) và ValueError (giá trị sai định dạng).

Dùng exception để xử lý input bất thường, không phải để che mọi bug. Hãy kiểm tra dữ liệu ở ranh giới nơi nó đi vào chương trình.

## 6. Đọc file an toàn

~~~python
from pathlib import Path

text = Path("notes.txt").read_text(encoding="utf-8")
print(text[:100])
~~~

Đường dẫn phụ thuộc vào current working directory. Nếu không tìm thấy file, hãy kiểm tra working directory và dùng đường dẫn tương đối rõ ràng từ project. Với dữ liệu CSV, hãy dùng module csv hoặc pandas thay vì tự tách dòng; dấu phẩy bên trong dấu ngoặc kép và trường bị thiếu khiến cách phân tích thủ công dễ sai.

## Bài luyện tập kèm đáp án

Viết function count_above(values, threshold) đếm các số lớn hơn ngưỡng. Với list [1, 5, 5, 8] và threshold bằng 5, đáp án là 1. Hãy nghĩ xem list rỗng nên cho kết quả gì: tự nhiên nhất là 0.

## Lỗi thường gặp

- Đặt tên biến là list, sum hoặc str, làm khuất các built-in function.
- Trộn số với chuỗi, chẳng hạn cộng số 3 với chuỗi "4" mà không chuyển kiểu.
- Viết một notebook cell khổng lồ thay vì chia thành các function nhỏ.
- Bắt mọi exception rồi tiếp tục chạy với dữ liệu đã hỏng.
