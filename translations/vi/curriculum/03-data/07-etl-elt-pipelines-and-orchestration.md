# Pipeline ETL/ELT và orchestration

[Xem bài gốc bằng tiếng Anh](../../../../curriculum/03-data/07-etl-elt-pipelines-and-orchestration.md)

## Mục tiêu học tập

Thiết kế data job có thể chạy lại an toàn, thể hiện dependency tường minh và phát hiện vấn đề chất lượng trước khi model học trên input sai.

## 1. ETL và ELT

**ETL** extract dữ liệu nguồn, transform rồi load kết quả đã chọn lọc vào storage. **ELT** load dữ liệu thô hoặc mới xử lý sơ bộ trước, sau đó transform trong platform đích. Cả hai đều cần provenance, kiểm soát truy cập, validation và cơ chế khôi phục. Lựa chọn phù hợp tùy volume, latency, quyền riêng tư, compute và platform.

## 2. Pipeline như directed acyclic graph

Workflow chạy theo lịch có thể gồm: ingest → kiểm tra schema → khử trùng lặp → join → tạo feature → kiểm tra chất lượng → publish. Directed acyclic graph (DAG) làm rõ dependency để scheduler có thể retry hoặc chạy song song các bước độc lập.

Giữ từng task nhỏ và có thể quan sát. Lưu trung gian lớn trong object/database storage; truyền reference và metadata giữa task thay vì nạp tất cả vào memory của scheduler.

## 3. Idempotency và xử lý tăng dần

Job **idempotent** nếu chạy hai lần vẫn tạo trạng thái dự kiến giống chạy một lần. Dùng key ổn định, transformation deterministic, ghi atomic và chiến lược thay thế/merge partition. Ghi watermark cho ingestion tăng dần. Retry không được tạo hàng trùng hoặc lặp side effect bên ngoài.

Phân biệt giao nhận at-least-once (event có thể lặp), at-most-once (event có thể mất) và tuyên bố exactly-once, thường cần semantics xử lý/lưu trữ phối hợp. Vẫn nên thiết kế deduplication và reconciliation dù platform có bảo đảm mạnh.

## 4. Quality gate và schema evolution

Kiểm tra cột/type bắt buộc, tính duy nhất, miền giá trị, tỷ lệ null, category, độ phủ thời gian, freshness và thay đổi số hàng. Định nghĩa điều kiện nào làm job fail, điều kiện nào tạo warning để điều tra. Version hóa và review schema change; thêm field tùy chọn khác với đổi tên field bắt buộc.

Không publish dataset ghi chưa xong. Ghi ra vị trí tạm/versioned, kiểm tra rồi mới đánh dấu version mới là hiện hành một cách atomic nếu storage hỗ trợ.

## 5. Lineage, observability và khôi phục

Ghi source version, code revision, run ID, partition input/output, số lượng, thời lượng và kết quả validation. Khi thất bại, giữ log và metadata một phần, báo người phụ trách và định nghĩa khi nào retry, backfill hoặc rollback. Pipeline âm thầm bỏ qua một ngày có thể làm thiếu training data mà lâu sau mới phát hiện.

## 6. Tạo feature

Tách định nghĩa feature tái sử dụng khỏi code notebook tùy hứng. Kiểm tra transformation khi training và serving nhất quán. Với feature có dữ liệu đến trễ, dùng point-in-time join để mỗi hàng train chỉ thấy giá trị đã có tại timestamp dự đoán.

## Thực hành

Thiết kế pipeline hằng ngày ingest order. Chỉ rõ idempotency key, watermark, chính sách event đến muộn, quality gate, version đầu ra, người nhận alert và quy trình backfill. Mô tả cách retry job tránh thêm order trùng.

## Lỗi thường gặp

- Cho rằng scheduler retry luôn an toàn dù các bước không idempotent.
- Ghi dataset một phần vào vị trí consumer đang đọc.
- Chỉ giám sát job success mà không theo dõi freshness hay volume dữ liệu.
- Tạo lại historical feature bằng giá trị hiện tại gây time leakage.
