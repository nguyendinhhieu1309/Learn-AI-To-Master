# Đệ quy và độ phức tạp thuật toán

[Xem bài gốc bằng tiếng Anh](../../../../curriculum/02-python/07-recursion-and-complexity.md)

Algorithm là quy trình hữu hạn, không nhập nhằng để giải quyết một bài toán. Complexity mô tả tài nguyên cần dùng tăng trưởng ra sao khi input tăng.

## Mục tiêu học tập

- Ước lượng độ phức tạp thời gian và bộ nhớ phụ.
- Đọc các lớp Big-O phổ biến và so sánh tốc độ tăng trưởng.
- Phân tích loop, nested loop và phép chia để trị.
- Viết recursive function với base case đúng.

## 1. Big-O đo lường gì

Gọi n là kích thước input. Big-O mô tả cận trên tiệm cận của tốc độ tăng trưởng, bỏ qua số giây phụ thuộc máy và các hệ số hằng. Đây không phải benchmark: hai chương trình O(n) có thể có chi phí hằng số rất khác nhau.

Các lớp tăng trưởng phổ biến, từ chậm đến nhanh:

| Complexity | Ví dụ thường gặp | Khi n gấp đôi |
| --- | --- | --- |
| O(1) | truy cập list theo index hoặc tra cứu dictionary trung bình | gần như không đổi |
| O(log n) | binary search | tăng khoảng một bước |
| O(n) | duyệt mọi phần tử | tăng gấp đôi |
| O(n log n) | comparison sort có bảo đảm tốt | tăng hơn gấp đôi một chút |
| O(n^2) | so sánh mọi cặp | tăng khoảng bốn lần |
| O(2^n) | liệt kê mọi tập con | gần như bình phương |
| O(n!) | liệt kê mọi hoán vị | tăng cực nhanh |

Khi n lớn, tốc độ tăng trưởng thấp thường quan trọng hơn cải thiện nhỏ về hằng số.

## 2. Đếm thao tác và rút gọn

~~~python
def find_value(values, target):
    for value in values:
        if value == target:
            return True
    return False
~~~

Loop có thể phải kiểm tra mọi phần tử: thời gian O(n). Bộ nhớ phụ là O(1).

~~~python
def print_all_pairs(values):
    for left in values:
        for right in values:
            print(left, right)
~~~

Nested loop thực hiện n nhân n thao tác: O(n^2). Nếu hai loop lần lượt xử lý input kích thước n và m, complexity có thể là O(nm), không phải O(n^2).

Bỏ qua hằng số và số hạng bậc thấp khi xét lớp tiệm cận: 5n+20 là O(n), n^2+3n là O(n^2). Khi cần, hãy nói rõ đang xét worst-case, average-case hay best-case.

## 3. Độ phức tạp thời gian và bộ nhớ

Time complexity ước lượng tốc độ tăng của số thao tác. Space complexity ước lượng tốc độ tăng của bộ nhớ. Tính cả vùng nhớ phụ và call stack đệ quy, nhưng phân biệt chúng với bộ nhớ của input có sẵn.

Algorithm sao chép list n phần tử dùng O(n) bộ nhớ phụ. Loop có vài biến đếm dùng O(1) bộ nhớ phụ. Chuỗi lời gọi đệ quy sâu n dùng O(n) call-stack space.

## 4. Chi phí khấu hao (amortized cost)

Thêm phần tử vào Python list có amortized complexity O(1): phần lớn lần thêm tốn thời gian hằng số, nhưng thỉnh thoảng phải tăng capacity và sao chép phần tử. Trung bình trên chuỗi thao tác dài, mỗi lần thêm vẫn có chi phí hằng số. Điều này không có nghĩa mọi lần thêm riêng lẻ đều tốn chính xác O(1).

## 5. Đệ quy

Recursive function tự gọi lại trên bài toán nhỏ hơn. Nó cần:

1. base case để dừng đệ quy;
2. bước tiến gần đến base case;
3. quy tắc kết hợp kết quả.

~~~python
def factorial(n: int) -> int:
    if n < 0:
        raise ValueError("n must be non-negative")
    if n <= 1:
        return 1
    return n * factorial(n - 1)
~~~

Function này tốn O(n) thời gian và O(n) bộ nhớ call stack. Python giới hạn độ sâu đệ quy; với bài toán có độ sâu tuyến tính lớn, hãy dùng cách lặp hoặc stack tường minh.

## 6. Chia để trị

Binary search loại bỏ một nửa đoạn đã sắp xếp ở mỗi bước, cần O(log n) phép so sánh. Merge sort chia input, sắp xếp hai nửa rồi trộn lại: recurrence là T(n)=2T(n/2)+O(n), cho thời gian O(n log n).

Mẫu “chia đôi input” thường gợi ý đệ quy logarithmic, nhưng vẫn phải tính công việc ở mỗi tầng và chi phí ghép kết quả.

## Bài luyện tập kèm đáp án

1. Complexity của việc duyệt một list rồi duyệt một list khác dài n và m là bao nhiêu?
2. Complexity của hai nested loop trên cùng n phần tử là bao nhiêu?
3. Bộ nhớ stack phụ của factorial(n) là bao nhiêu?
4. Vì sao list append được gọi là amortized O(1)?

### Đáp án

1. O(n+m).
2. O(n^2).
3. O(n) recursive frame.
4. Hầu hết lần append tốn chi phí hằng số; các lần resize đắt được phân bổ trên nhiều thao tác.
