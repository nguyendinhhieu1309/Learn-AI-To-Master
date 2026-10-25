# Nền tảng Python và Computing

[Xem trang gốc bằng tiếng Anh](../../../../curriculum/02-python/README.md)

Ví dụ code trong module này dùng Python 3.10 trở lên.

## Kết quả học tập

- Viết chương trình Python nhỏ, dễ đọc và function có thể tái sử dụng.
- Hiểu object-oriented programming và chọn cấu trúc tích hợp phù hợp với tác vụ.
- Phân tích time/space complexity và cài đặt các data structure, algorithm cốt lõi.
- Làm việc với file, bảng dữ liệu, notebook, package và version control.
- Gỡ lỗi phổ biến và làm cho kết quả có thể tái tạo.

## Thứ tự học gợi ý

1. [Python cơ bản cho người học AI](01-python-basics.md)
2. [Ngôn ngữ cốt lõi: type, string và collection](04-python-core-language.md)
3. [Function, module, file và lỗi](05-functions-modules-files-and-errors.md)
4. [Array và bảng với NumPy, pandas](02-arrays-and-tables.md)
5. [Lập trình hướng đối tượng](06-object-oriented-programming.md)
6. [Đệ quy và độ phức tạp thuật toán](07-recursion-and-complexity.md)
7. [Data structure tích hợp và chi phí](08-built-in-data-structures.md)
8. [Linked list, stack và queue](09-linked-lists-stacks-and-queues.md)
9. [Hash table, tìm kiếm và sắp xếp](10-hash-tables-searching-and-sorting.md)
10. [Tree, binary search tree và heap](11-trees-bst-and-heaps.md)
11. [Graph, traversal và đường đi ngắn nhất](12-graphs-and-graph-algorithms.md)
12. [Mẫu tư duy giải bài toán](13-algorithmic-problem-solving-patterns.md)
13. [Lab Python tái lập](03-reproducible-python-lab.md)
14. [Bài tập Python, DSA kèm lời giải](PROBLEM_SET_WITH_SOLUTIONS.md)
15. Sau đó học Git, environment và software engineering trong [AI Engineering](../11-ai-engineering/README.md).

## Thực hành

- Viết function tóm tắt list số và xử lý trường hợp list rỗng.
- Cài đặt thao tác linked list, BST và graph sau khi phân tích complexity.
- Giải thích binary search khác linear search thế nào và khi nào binary search hợp lệ.
- Đọc file text hoặc CSV, kiểm tra từng hàng và báo lỗi input sai định dạng rõ ràng.
- Chuyển chuỗi thao tác trong notebook thành các function có thể tái sử dụng.
- Dùng Git commit để ghi lại thay đổi nhỏ và xem diff.
- Hoàn thành [lab kiểm tra CSV](03-reproducible-python-lab.md), gồm cả test cho trường hợp lỗi.

## Dự án checkpoint

Tạo chương trình command-line hoặc notebook nạp dataset nhỏ, báo kích thước và thống kê mô tả cơ bản, xử lý một giá trị thiếu rồi lưu bản đã làm sạch. Ghi hướng dẫn để người khác tái tạo kết quả.

## Lỗi thường gặp cần để ý

- Dùng notebook thay cho việc tổ chức code có thể tái sử dụng.
- Cài package toàn cục rồi không theo dõi được environment.
- Bỏ qua shape, type và giá trị thiếu cho đến khi model training bị lỗi.
- Tạo commit lớn khiến khó biết điều gì đã thay đổi.
- Mặc định dùng linked list hoặc tự viết sort dù list, dict, set, deque hay built-in sort của Python phù hợp hơn.
