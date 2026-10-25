# Cấu trúc dữ liệu tích hợp và chi phí thao tác

[Xem bài gốc bằng tiếng Anh](../../../../curriculum/02-python/08-built-in-data-structures.md)

Chọn cấu trúc dựa vào các thao tác mà chương trình cần thực hiện. Built-in của Python đã được tối ưu, thường nên ưu tiên hơn cấu trúc tự cài đặt.

## 1. List: mảng động

List lưu chuỗi có thứ tự và hỗ trợ truy cập theo index.

| Thao tác | Complexity thường gặp |
| --- | --- |
| Đọc hoặc gán theo index | O(1) |
| Tìm một giá trị | O(n) |
| Thêm vào cuối | amortized O(1) |
| Chèn hoặc xóa gần đầu | O(n), do phải dịch chuyển |
| Sắp xếp | O(n log n) trong worst case |

Dùng list cho batch, kết quả có thứ tự, stack và dữ liệu cần truy cập theo index.

## 2. Tuple và range

Tuple là record hoặc chuỗi có thứ tự nhưng bất biến. Range biểu diễn dãy số nguyên gọn nhẹ mà không cần tạo sẵn mọi giá trị.

~~~python
point = (3.5, 7.2)
for index in range(1_000_000):
    pass
~~~

Tính bất biến giúp tuple làm dictionary key được nếu mọi phần tử bên trong đều hashable.

## 3. Dictionary và set

Tra cứu, chèn và xóa trong dictionary và set trung bình là O(1), worst case là O(n) nếu va chạm bất thường. Dictionary ánh xạ key sang value; set chỉ lưu key.

~~~python
counts = {}
for label in ["cat", "dog", "cat"]:
    counts[label] = counts.get(label, 0) + 1
~~~

Dùng set cho membership và tính duy nhất. Key cần có phép so sánh bằng và hashing ổn định. Không thay đổi object theo cách làm đổi hash trong lúc object đang được lưu làm key.

## 4. Queue: dùng deque

Xóa phần tử đầu list khiến các phần tử còn lại dịch chuyển, tốn O(n). Dùng collections.deque để thao tác hiệu quả ở cả hai đầu:

~~~python
from collections import deque

queue = deque(["first"])
queue.append("second")
item = queue.popleft()
~~~

Thêm hoặc xóa ở mỗi đầu tốn O(1). Deque không hỗ trợ truy cập index tùy ý ở giữa một cách hiệu quả.

## 5. Priority queue: dùng heapq

Min-heap cung cấp phần tử nhỏ nhất. Thêm và lấy phần tử khỏi heap tốn O(log n); đọc phần tử nhỏ nhất tốn O(1).

~~~python
import heapq

tasks = [(4, "later"), (1, "urgent"), (3, "normal")]
heapq.heapify(tasks)
priority, task = heapq.heappop(tasks)
~~~

Dùng priority queue khi cần chọn phần tử kế tiếp theo độ ưu tiên thay vì thứ tự được thêm vào.

## 6. Chọn theo thao tác cần thiết

| Nhu cầu | Cấu trúc phù hợp để bắt đầu |
| --- | --- |
| Chuỗi có thứ tự và truy cập index ngẫu nhiên | list |
| Tra cứu theo key | dict |
| Membership nhanh và phần tử duy nhất | set |
| Thêm/xóa ở cả hai đầu | deque |
| Luôn lấy phần tử có priority nhỏ nhất hoặc lớn nhất | heapq |
| Nhóm giá trị cố định, bất biến | tuple |

Với dữ liệu rất nhỏ, lựa chọn có asymptotic complexity tốt nhất chưa chắc chạy nhanh nhất. Ưu tiên code rõ ràng; nếu hiệu năng quan trọng thì hãy đo.

## Bài luyện tập kèm đáp án

1. Vì sao xóa index 0 khỏi list có complexity O(n)?
2. Nên dùng cấu trúc nào làm hàng đợi trong BFS?
3. Nên dùng cấu trúc nào để lấy sự kiện có thời điểm sớm nhất?

### Đáp án

1. Các phần tử còn lại phải dịch sang trái.
2. deque.
3. Min-heap.
