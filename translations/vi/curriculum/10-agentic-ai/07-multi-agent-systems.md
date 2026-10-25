# Multi-Agent System

## Mục tiêu học tập

Chọn orchestration pattern, xác định ranh giới rõ giữa agent và đánh giá delegation có cải thiện kết quả đủ để bù độ phức tạp không.

## 1. Pattern phổ biến

- **Sequential pipeline:** các chuyên gia/stage cố định chạy theo thứ tự (extract → research → write → verify).
- **Router/handoff:** agent triage chọn một chuyên gia xử lý request.
- **Manager with agents-as-tools:** manager giữ quyền điều phối, gọi specialist làm subtask giới hạn rồi kết hợp kết quả.
- **Parallel fan-out/fan-in:** tác vụ độc lập chạy đồng thời; reducer kết hợp kết quả.
- **Collaborative loop:** agent lặp lại việc phê bình hoặc sửa kết quả của nhau; cần giới hạn iteration nghiêm ngặt và rubric khách quan.

## 2. Handoff so với agent-as-tool

Với **handoff**, quyền điều khiển chuyển sang specialist; specialist thường trở thành responder chính của nhánh đó. Với **agent-as-tool**, manager gọi specialist làm subtask nhưng vẫn chịu trách nhiệm kết quả gửi user. Chọn theo bên cần sở hữu câu trả lời cuối và mức context mỗi specialist cần.

## 3. Ranh giới và shared state

Cho mỗi specialist role hẹp, input/output schema tường minh và điều kiện thành công. Chỉ truyền context liên quan, không mặc định gửi cả cuộc hội thoại. Phân biệt evidence dùng chung không thay đổi với mutable task state. Quy định xử lý kết quả mâu thuẫn, thiếu, timeout và hoàn tất một phần.

Parallelism chỉ hữu ích khi công việc độc lập. Nếu hai agent ghi cùng resource hoặc phụ thuộc kết luận của nhau, cần coordination hoặc chạy tuần tự. Gộp phát hiện trùng lặp và giữ provenance.

## 4. Cost, latency và reliability

Mỗi agent có thể thêm model call, token, tool execution và failure point. Đo end-to-end task success, cost, latency, mức bất đồng và recovery so với baseline một agent/workflow. Giới hạn recursion và fan-out. Manager không nên gọi specialist lặp lại nếu thiếu stopping rule.

## 5. Evaluation và security

Kiểm tra routing sai, output specialist mâu thuẫn, nội dung retrieval độc hại, agent thất bại, action trùng và ranh giới permission. Thực thi authorization tại từng tool execution kể cả khi specialist được tin. Giữ audit trace cho biết agent nào đề xuất action nào và kết quả nào được sử dụng.

## Thực hành

Với technical-support assistant, so sánh (a) một agent có documentation search, (b) router với billing và technical specialist, (c) manager hỏi cả hai specialist lấy evidence. Nêu khi nào mỗi thiết kế phù hợp, failure mode và cách đo cost/routing chính xác.

## Lỗi thường gặp

- Thêm agent vì “multi-agent” nghe có vẻ tiên tiến.
- Cấp mọi agent mọi tool, secret và toàn bộ conversation history.
- Cho phép delegation loop không kiểm soát hoặc parallel call không giới hạn.
- Gộp câu trả lời mâu thuẫn mà không giữ nguồn hoặc yêu cầu review.
