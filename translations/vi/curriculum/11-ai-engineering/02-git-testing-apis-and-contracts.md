# Git, Testing, API và Data Contract

## Mục tiêu học tập

Review thay đổi bằng Git, chọn test phù hợp cho AI software và đưa hành vi model ra interface rõ ràng.

## 1. Git như lịch sử công việc

Git lưu snapshot (commit) để nhóm review thay đổi có phạm vi rõ. Kiểm tra git status và xem diff trước commit. Viết commit message giải thích thay đổi. Branch và pull request hỗ trợ làm song song/review; không commit credential thô hoặc dataset bị hạn chế.

## 2. Các tầng kiểm thử

- **Unit test:** function tất định như parse ngày, feature transform, threshold logic và permission check.
- **Contract/schema test:** xác minh cột bắt buộc, type, unit và output shape.
- **Integration test:** kiểm tra storage, model loading, retrieval hoặc tool adapter kết nối đúng.
- **End-to-end test:** chạy tác vụ đại diện qua toàn application.
- **Model evaluation:** đo quality trên dataset/task suite có version; đây là evidence thống kê, không thay software test.

Model test nên dùng case đại diện và tolerance khi floating-point output không giống hệt. Kiểm tra thuộc tính quan trọng (shape, range hợp lệ, latency budget hoặc mức đổi score chấp nhận được), không assert một số thập phân mong manh cho mọi neural output.

## 3. Cơ bản về HTTP API

HTTP request gồm method, path, header và body tùy chọn. Prediction API thường nhận JSON, validate, tiền xử lý/inference rồi trả JSON với status phù hợp. POST thường dùng cho prediction body; GET thường dùng để lấy resource. Trả error tường minh khi input sai, model không khả dụng, timeout hoặc lỗi nội bộ. Đừng trả prediction thành công nếu model không chạy.

Định nghĩa contract có version: field, type, unit, tập category, giá trị tùy chọn, nghĩa output label, ý nghĩa confidence và cách báo lỗi. Backward compatibility quan trọng khi client deploy độc lập nhau.

## 4. Retry, timeout và idempotency

Network call có thể lỗi hoặc timeout. Thêm timeout cho external call; chỉ retry khi phù hợp, có backoff và retry limit. Action không idempotent (ví dụ tạo đơn hàng) có thể bị lặp ngoài ý muốn nếu timeout xảy ra sau khi action thành công. Dùng idempotency key và kiểm tra action status trước retry.

## 5. Security ở interface

Xác thực client, authorize từng operation, rate-limit abuse, validate payload size/field và tránh trả stack trace cho caller không tin cậy. Input validation giảm dữ liệu sai định dạng, không làm output model trở nên chính xác. Với endpoint dùng LLM, cần tính cả prompt injection, output validation và nội dung retrieval không tin cậy.

## Thực hành

Viết API contract cho image classifier: format được nhận, kích thước/byte tối đa, preprocessing, class output, ý nghĩa confidence, timeout và error code. Thiết kế một unit test, một contract test và một end-to-end evaluation case.

## Lỗi thường gặp

- Coi một assertion test accuracy là evaluation model đầy đủ.
- Retry action không thể hoàn tác mà không có idempotency strategy.
- Trả secret nội bộ hoặc exception trace cho caller.
- Đổi label mapping output mà không đổi contract version.
