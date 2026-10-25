# Python, OOP và DSA: Bộ bài tập kèm lời giải

[Xem bài gốc bằng tiếng Anh](../../../../curriculum/02-python/PROBLEM_SET_WITH_SOLUTIONS.md)

Hãy làm lần lượt các câu hỏi. Với câu hỏi thuật toán, nêu giả định và phân tích cả thời gian lẫn bộ nhớ phụ.

## 1. Slicing

Với values=[10,20,30,40,50], kết quả của values[1:4] là gì?

### Lời giải

[20,30,40]. Index bắt đầu được lấy, index kết thúc bị loại trừ.

## 2. Aliasing

Hai tên cùng trỏ đến một list. Một tên thêm phần tử. Tên còn lại có thấy phần tử đó không?

### Lời giải

Có. Phép gán tạo tên thứ hai cho cùng một object có thể thay đổi. Nếu cần list độc lập, hãy tạo bản sao.

## 3. Đếm frequency

Chọn cấu trúc dữ liệu và complexity để đếm nhãn lặp trong n record.

### Lời giải

Dùng dictionary ánh xạ label sang count. Thời gian kỳ vọng O(n), bộ nhớ phụ O(k), với k là số label duy nhất.

## 4. Mutable default

Vì sao def add(x, items=[]) có rủi ro, và pattern nào tránh state dùng chung?

### Lời giải

List mặc định được tạo một lần rồi dùng lại giữa các lần gọi. Dùng items=None, rồi tạo list mới bên trong function khi items là None.

## 5. Return hay print

Function chuẩn hóa dữ liệu lại in giá trị sau biến đổi. Vì sao cách này khó tái sử dụng hơn trả về giá trị?

### Lời giải

In ra là phần trình bày, function khác khó xử lý tiếp. Return hỗ trợ ghép bước, kiểm thử và lưu theo nhiều định dạng.

## 6. Dùng OOP hay function?

Một function tính IoU cho hai bounding box và không lưu state. Có nhất thiết phải viết class không?

### Lời giải

Không. Pure function đơn giản hơn. Dùng class khi state và hành vi có ý nghĩa cần đi cùng nhau, hoặc dùng data class để biểu diễn box.

## 7. Inheritance hay composition?

Một detector dùng image preprocessor có thể thay thế. Detector có nên kế thừa từ preprocessor không?

### Lời giải

Thường là không. Detector có một preprocessor; composition thể hiện đúng quan hệ và giúp thay thành phần dễ dàng.

## 8. Complexity của các loop tách biệt

Một loop xử lý n hàng; loop thứ hai xử lý m cột. Tổng thời gian là bao nhiêu?

### Lời giải

O(n+m), không phải O(nm), vì các loop chạy nối tiếp nhau.

## 9. Complexity của so sánh cặp

Nested loop so sánh mọi cặp trong list n phần tử có complexity thời gian nào?

### Lời giải

O(n^2). Có thể thực hiện n bình phương phép so sánh, hoặc n(n-1)/2 khi xét các cặp khác nhau không phân biệt thứ tự; cả hai đều rút gọn thành O(n^2).

## 10. Linear search và binary search

Một list có một triệu giá trị đã sắp xếp. Chi phí so sánh tiệm cận của linear search và binary search là bao nhiêu?

### Lời giải

Linear search là O(n), có thể tới khoảng một triệu lần kiểm tra. Binary search là O(log2 n), khoảng 20 bước chia đôi. Binary search yêu cầu dữ liệu đã sắp xếp và truy cập ngẫu nhiên.

## 11. Queue bằng list hay deque

Vì sao dùng list.pop(0) lặp lại không phải cách cài queue tốt?

### Lời giải

Mỗi lần xóa đều dịch chuyển các phần tử còn lại, tốn O(n). deque.popleft tốn O(1).

## 12. Complexity của linked list

Append tốn bao nhiêu khi có và không có tail pointer?

### Lời giải

Có tail pointer thì O(1); không có thì phải duyệt đến node cuối, tốn O(n). Tìm theo value tốn O(n) trong cả hai trường hợp.

## 13. Chi phí hash table

Complexity tra cứu dictionary kỳ vọng và worst case là bao nhiêu?

### Lời giải

Kỳ vọng/trung bình O(1), worst case O(n). Hashing không đảm bảo thời gian hằng số với mọi kiểu collision.

## 14. Stable sorting

Hai record có cùng score. Stable sort bảo toàn điều gì?

### Lời giải

Thứ tự tương đối của chúng trong input. Điều này hữu ích khi sắp xếp nhiều key qua từng giai đoạn.

## 15. BST suy biến

Chèn các giá trị 1,2,3,4 theo thứ tự đó vào BST không cân bằng. Hình dạng và complexity tìm kiếm có thể ra sao?

### Lời giải

Tree thành một chuỗi nghiêng sang phải. Height là O(n), nên search và insert có thể suy giảm thành O(n).

## 16. Xóa khỏi BST

Xóa node BST có hai node con như thế nào?

### Lời giải

Thay nó bằng in-order successor hoặc predecessor, rồi xóa node thay thế khỏi vị trí cũ. Node thay thế có tối đa một node con.

## 17. Heap hay list đã sắp xếp

Chương trình liên tục thêm task và lấy task có priority nhỏ nhất. Vì sao heap có thể phù hợp hơn?

### Lời giải

Thêm và lấy khỏi heap tốn O(log n), đọc giá trị nhỏ nhất tốn O(1). Chèn vào list được giữ sắp xếp có thể cần dịch chuyển O(n) phần tử.

## 18. BFS và đường đi ngắn nhất

Traversal graph nào tìm route có ít edge nhất trong graph không trọng số?

### Lời giải

BFS dùng FIFO queue. Nó duyệt vertex theo khoảng cách tăng dần từ source.

## 19. Biểu diễn graph

Biểu diễn nào nên dùng làm mặc định cho graph thưa có hàng triệu cặp vertex tiềm năng nhưng tương đối ít edge?

### Lời giải

Adjacency list: bộ nhớ O(V+E), thay vì O(V^2) cho adjacency matrix.

## 20. Giới hạn của Dijkstra

Vì sao Dijkstra yêu cầu trọng số edge không âm?

### Lời giải

Quy tắc tham lam xem khoảng cách nhỏ nhất hiện biết là đã chốt. Edge âm xuất hiện sau có thể giảm khoảng cách đó và làm quy tắc sai. Dùng thuật toán hỗ trợ edge âm và phát hiện negative cycle.

## 21. Tiền điều kiện của sliding window

Vì sao ví dụ shortest-subarray chỉ thu hẹp cạnh trái khi mọi giá trị không âm?

### Lời giải

Với giá trị không âm, bỏ phần tử bên trái không thể làm tổng tăng. Số âm phá vỡ tính đơn điệu này nên chứng minh và thuật toán không còn áp dụng.

## 22. Review thuật toán

Trước khi tối ưu lời giải chậm, cần xác định điều gì?

### Lời giải

Contract input/output đúng, constraint, lời giải đơn giản đúng, ước lượng complexity và edge case đã kiểm tra. Sau đó đo bottleneck rồi chọn representation hoặc algorithm tốt hơn.

## Bài thử thách

Xây dựng route planner nhỏ trên weighted graph. Yêu cầu: trả về khoảng cách ngắn nhất và route, từ chối trọng số âm, xử lý khi không có route, đồng thời ghi rõ cách biểu diễn graph, complexity và cách xử lý trường hợp bằng nhau.
