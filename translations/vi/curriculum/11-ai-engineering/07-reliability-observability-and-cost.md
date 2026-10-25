# Reliability, Observability và Cost

## Mục tiêu học tập

Dùng log, metric và trace để hiểu AI application; định nghĩa service objective; nối alert với phản hồi có thể thực hiện.

## 1. Log, metric và trace

- **Log** ghi sự kiện rời rạc (validation thất bại, model được load, tool timeout).
- **Metric** tổng hợp giá trị theo thời gian (request/giây, p95 latency, error rate, token use, queue depth).
- **Trace** nối các bước trong một request (API → retrieval → model → tool → response).

Dùng correlation ID để theo request qua nhiều service. Redact hoặc tránh lưu nội dung cá nhân/secret. Đặt retention và access control phù hợp dữ liệu.

## 2. Service-level indicator và objective

Service-level indicator (SLI) là thuộc tính đo được như availability, error rate hoặc p95 latency. Service-level objective (SLO) đặt mục tiêu trong khoảng thời gian. Chọn indicator phản ánh trải nghiệm user chứ không chỉ process còn sống. Với model application, thêm tín hiệu quality hoặc safe completion nếu có label/audit.

## 3. Monitoring model/application

Theo dõi thay đổi feature/schema, missingness, prediction distribution, confidence/calibration khi có nghĩa, subgroup performance, delayed label, retrieval hit rate, citation support, tool failure rate và model/provider version. Input drift đơn lẻ không chứng minh quality mất; input ổn định cũng không đảm bảo quality. Kết hợp tín hiệu với audit hoặc label.

## 4. Alert và incident response

Alert cần owner, severity và hành động. Định nghĩa runbook cho provider outage, model-load failure, latency tăng đột ngột, data hỏng, action không an toàn hoặc quality regression. Gồm rollback, degraded mode, thông tin tới user và incident note. Tránh alert nhiễu khiến operator quen bỏ qua.

## 5. Cost model

Ước tính tổng chi phí: model input/output, embedding, retrieval, database, GPU/CPU time, storage, network, monitoring, retry và human review. Theo dõi cost trên mỗi tác vụ thành công và theo workload/tenant khi phù hợp. Budget control giới hạn token, step, concurrency và retention. Sau mỗi tối ưu cần xem lại quality; output rẻ nhất vô ích nếu không hoàn thành tác vụ.

## 6. Reliability pattern

Dùng retry có giới hạn với exponential backoff và jitter cho lỗi tạm thời; idempotency key cho side effect; circuit breaker cho dependency lỗi; backpressure khi overload. Tránh retry storm. Cho user và log thấy rõ degraded/fallback behavior.

## Thực hành

Với LLM support assistant, định nghĩa SLO cho response latency và availability, hai application-quality indicator, fallback khi provider outage, dashboard cost-per-success và một high-severity alert runbook.

## Lỗi thường gặp

- Chỉ monitor CPU/server uptime mà bỏ qua task failure.
- Alert theo thống kê nhưng không có response owner.
- Mặc định log toàn bộ prompt và tài liệu retrieval.
- Dùng automatic retry làm overload nặng hơn hoặc lặp side effect.
