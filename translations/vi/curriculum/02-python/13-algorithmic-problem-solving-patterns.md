# Mẫu tư duy giải bài toán thuật toán

[Xem bài gốc bằng tiếng Anh](../../../../curriculum/02-python/13-algorithmic-problem-solving-patterns.md)

Bài này biến đề bài thành thuật toán rõ ràng. Các pattern là điểm xuất phát có thể tái sử dụng, không phải quy tắc cần ép vào mọi bài toán.

## 1. Quy trình làm bài có thể lặp lại

1. Diễn đạt lại tác vụ và định nghĩa chính xác input/output.
2. Tự giải ví dụ nhỏ bằng tay, gồm input rỗng và phần tử trùng.
3. Xác định constraint: kích thước input, đã sắp xếp chưa, bộ nhớ và có cập nhật hay không.
4. Trước tiên viết thuật toán đúng đơn giản nhất.
5. Ước lượng thời gian và bộ nhớ phụ.
6. Kiểm tra edge case và chứng minh invariant quan trọng.
7. Cài đặt, kiểm tra và so sánh với bản brute-force trên input nhỏ.

“Khó” không phải complexity class. Độ khó thường đến từ việc nhận ra cách biểu diễn phù hợp hoặc chứng minh tính đúng; Big-O mô tả tốc độ tăng sau khi đã chọn thuật toán.

## 2. Frequency map

Đếm số lần xuất hiện bằng dictionary trong thời gian kỳ vọng O(n):

~~~python
def frequencies(values):
    count = {}
    for value in values:
        count[value] = count.get(value, 0) + 1
    return count
~~~

Dùng cho phát hiện trùng lặp, anagram, đếm category và ghép theo key.

## 3. Two pointers

Với list đã sắp xếp, có thể đưa hai con trỏ vào trong mà không phải kiểm tra mọi cặp:

~~~python
def has_pair_sum(sorted_values: list[int], target: int) -> bool:
    left, right = 0, len(sorted_values) - 1
    while left < right:
        total = sorted_values[left] + sorted_values[right]
        if total == target:
            return True
        if total < target:
            left += 1
        else:
            right -= 1
    return False
~~~

Thời gian O(n), bộ nhớ phụ O(1). Nếu phải sắp xếp trước thì tốn O(n log n); đừng quên chi phí này khi input ban đầu chưa có thứ tự.

## 4. Sliding window

Với một đoạn liên tiếp, duy trì một cửa sổ và cập nhật thống kê khi hai đầu di chuyển. Với giá trị không âm, đoạn code sau tìm segment ngắn nhất có tổng đạt ngưỡng:

~~~python
def shortest_at_least(target: int, values: list[int]) -> int:
    left = 0
    total = 0
    best = len(values) + 1
    for right, value in enumerate(values):
        total += value
        while total >= target:
            best = min(best, right - left + 1)
            total -= values[left]
            left += 1
    return 0 if best > len(values) else best
~~~

Logic cửa sổ này phụ thuộc vào việc các giá trị không âm. Nếu có số âm, mở rộng hoặc thu hẹp cửa sổ không còn làm tổng thay đổi đơn điệu; cần phương pháp khác.

## 5. Prefix sum

Định nghĩa prefix[i] là tổng i giá trị đầu tiên. Tổng các giá trị từ index left đến right bao gồm cả hai đầu là prefix[right+1]-prefix[left].

~~~python
def prefix_sums(values: list[int]) -> list[int]:
    prefix = [0]
    for value in values:
        prefix.append(prefix[-1] + value)
    return prefix
~~~

Tạo prefix trong O(n); trả lời từng câu hỏi tổng trên một khoảng trong O(1), dùng O(n) bộ nhớ.

## 6. Binary search trên điều kiện

Binary search có thể tìm vị trí đầu tiên mà một điều kiện đơn điệu chuyển từ false sang true. Nêu invariant và cập nhật giới hạn cẩn thận. Điều kiện phải giữ tính đơn điệu trong vùng tìm kiếm.

Ví dụ: tìm giá trị đầu tiên >= target trong array đã sắp xếp, hoặc capacity nhỏ nhất có thể xử lý xong một nhóm job. Đây không phải đoán midpoint rồi bỏ một nửa mà không có chứng minh.

## 7. Stack và monotonic stack

Stack hữu ích cho cấu trúc lồng nhau và bài toán “phần tử lớn hơn/nhỏ hơn kế tiếp”. Duy trì các giá trị tăng hoặc giảm; loại bỏ phần tử không còn có thể là đáp án. Mỗi phần tử được push và pop tối đa một lần, nên toàn thuật toán thường là O(n), dù trong for có while.

Dùng pattern này cho ghép ngoặc, next greater element và tìm hình chữ nhật lớn nhất trong histogram.

## 8. Graph search và dynamic programming

Dùng BFS để tìm path ngắn nhất theo số edge trong graph không trọng số. Dùng DFS cho reachability, component và backtracking. Dùng dynamic programming khi bài toán có subproblem lặp lại và optimal substructure; hãy định nghĩa state và recurrence trước khi viết code.

## Bài luyện tập kèm đáp án

1. Vì sao phương pháp two-pointer tìm cặp có tổng bằng target yêu cầu input được sắp xếp?
2. Vì sao ví dụ sliding-window không dùng được với số âm tùy ý?
3. Vì sao nested while vẫn có thể chạy O(n) tổng thể trong monotonic stack?
4. Trước khi cài đặt dynamic programming, cần viết ra điều gì?

### Đáp án

1. Chiều thay đổi tổng cho biết có thể di chuyển con trỏ nào một cách an toàn.
2. Xóa một giá trị có thể làm tổng tăng, nên invariant không còn đúng.
3. Mỗi phần tử được thêm một lần và bỏ một lần, nên tổng thao tác con trỏ/stack là tuyến tính.
4. Ý nghĩa state, base case, recurrence, thứ tự phụ thuộc và cách lấy đáp án.
