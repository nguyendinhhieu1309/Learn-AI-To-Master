# Hash table, tìm kiếm và sắp xếp

[Xem bài gốc bằng tiếng Anh](../../../../curriculum/02-python/10-hash-tables-searching-and-sorting.md)

Bài này giải thích cách tìm và sắp xếp dữ liệu, so sánh chi phí thuật toán và dùng hashing để tra cứu theo key nhanh.

## 1. Hash table

Hash table ánh xạ key tới một vị trí bằng hash function. Nhiều key khác nhau có thể được ánh xạ tới cùng vị trí; đây là collision. Implementation cần phát hiện collision và có thể resize khi bảng quá đầy.

Dictionary và set của Python dựa trên hash table. Tra cứu, chèn và xóa trung bình là O(1), nhưng worst case có thể là O(n). Key phải giữ phép so sánh bằng và hash ổn định khi được lưu. String, số và tuple chứa các giá trị hashable là key phổ biến.

~~~python
word_counts = {}
for word in ["ai", "data", "ai"]:
    word_counts[word] = word_counts.get(word, 0) + 1
~~~

Hash table không phải sorted map. Nếu cần duyệt theo thứ tự, hãy sắp xếp key hoặc dùng cấu trúc có thứ tự phù hợp.

## 2. Linear search

Linear search kiểm tra từng giá trị:

~~~python
def linear_search(values: list[int], target: int) -> int:
    for index, value in enumerate(values):
        if value == target:
            return index
    return -1
~~~

Thời gian là O(n), bộ nhớ phụ là O(1). Cách này dùng được với dữ liệu chưa sắp xếp và có thể là lựa chọn đơn giản nhất cho input nhỏ.

## 3. Binary search

Binary search liên tục chia đôi vùng tìm kiếm đã được sắp xếp. Cần O(log n) phép so sánh và O(1) bộ nhớ phụ trong implementation lặp. Input phải được sắp xếp theo đúng thứ tự đang dùng để so sánh.

~~~python
def binary_search(values: list[int], target: int) -> int:
    left, right = 0, len(values) - 1
    while left <= right:
        middle = left + (right - left) // 2
        if values[middle] == target:
            return middle
        if values[middle] < target:
            left = middle + 1
        else:
            right = middle - 1
    return -1
~~~

Giá trị trả về là một index khớp hoặc -1 nếu không tìm thấy. Muốn tìm phần tử trùng đầu tiên, hãy tiếp tục tìm bên trái sau khi gặp một kết quả. Lỗi thường gặp là dùng vùng đóng/mở không nhất quán, gây vòng lặp vô hạn hoặc bỏ qua giá trị.

## 4. Thuật toán sắp xếp

| Algorithm | Best time | Average time | Worst time | Bộ nhớ phụ | Stable? |
| --- | --- | --- | --- | --- | --- |
| Bubble sort | O(n) nếu dừng sớm | O(n^2) | O(n^2) | O(1) | Thường ổn định |
| Selection sort | O(n^2) | O(n^2) | O(n^2) | O(1) | Thường không |
| Insertion sort | O(n) | O(n^2) | O(n^2) | O(1) | Có |
| Merge sort | O(n log n) | O(n log n) | O(n log n) | O(n) | Có |
| Quick sort | O(n log n) | O(n log n) | O(n^2) | O(log n) stack trung bình | Thường không |

“Stable” nghĩa là các phần tử có key bằng nhau giữ nguyên thứ tự tương đối trong input. Tính ổn định hữu ích khi sắp xếp record theo nhiều key qua các bước.

~~~python
records = [{"name": "A", "score": 8}, {"name": "B", "score": 5}]
by_score = sorted(records, key=lambda row: row["score"])
~~~

Built-in sort của Python ổn định và được tối ưu cao. Dùng nó trong code ứng dụng; hãy tự cài đặt classic sort để học cách chúng hoạt động và complexity.

## 5. Chọn phương án tìm kiếm/sắp xếp

- Chuỗi nhỏ, chưa sắp xếp: linear search.
- Nhiều lần tra cứu trên dữ liệu tĩnh: sắp xếp một lần rồi binary search, hoặc tạo set/dictionary nếu cần tra cứu key chính xác.
- Cần đầu ra có thứ tự: sắp xếp theo key rõ ràng.
- Cần lấy min/max lặp lại: dùng heap thay vì sắp xếp lại sau mỗi lần cập nhật.

Khi so sánh, hãy tính cả chi phí preprocessing. Sắp xếp n phần tử tốn O(n log n); chỉ đáng làm cho nhiều lần tìm khi tổng workload hợp lý.

## Bài luyện tập kèm đáp án

1. Binary search đòi hỏi điều kiện nào?
2. Worst-case time của quick sort là bao nhiêu?
3. Stable sort giữ lại điều gì?
4. Vì sao dictionary lookup không thể thay thế mọi binary search?

### Đáp án

1. Chuỗi cần tìm phải được sắp xếp theo cùng thứ tự so sánh.
2. O(n^2) với pivot không thuận lợi.
3. Thứ tự tương đối của các phần tử có key bằng nhau.
4. Binary search dựa vào thứ tự và có thể trả lời câu hỏi về khoảng; dictionary lookup dành cho key chính xác, không giữ ngữ nghĩa tìm kiếm có thứ tự.
