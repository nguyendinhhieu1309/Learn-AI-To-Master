# Batch, streaming và tính nhất quán của feature

[Xem bài gốc bằng tiếng Anh](../../../../curriculum/03-data/08-batch-streaming-and-feature-consistency.md)

## Mục tiêu học tập

Chọn batch hay streaming dựa trên nhu cầu sản phẩm, suy luận về event trễ/trùng và giữ feature nhất quán giữa model training với dự đoán online.

## 1. Batch và streaming

Batch processing xử lý dataset có giới hạn theo lịch: báo cáo hằng đêm hoặc refresh model hằng ngày. Streaming xử lý event liên tục hoặc theo window nhỏ. Streaming giảm latency nhưng tăng độ phức tạp: thứ tự, duplicate, backpressure, checkpoint, dữ liệu trễ và replay.

Chọn cách đơn giản nhất đáp ứng freshness cần thiết. Yêu cầu gần real-time nên được nêu bằng chỉ số đo được (ví dụ p95 freshness dưới ngưỡng xác định) trước khi thiết kế streaming platform.

## 2. Event time và processing time

**Event time** là lúc sự kiện xảy ra; **processing time** là lúc hệ thống nhận nó. Event có thể đến trễ hoặc sai thứ tự. Windowed aggregate cần chính sách cho dữ liệu muộn và việc sửa kết quả. Feature forecasting chỉ được dùng event đã nhận trước cutoff quyết định, kể cả khi timestamp sự kiện sớm hơn nhưng event đến sau.

## 3. Training/serving skew

Training/serving skew xảy ra khi feature được tính khác nhau trong dữ liệu lịch sử và inference trực tiếp. Ví dụ: lệch timezone, mapping category khác, nguồn field không có sẵn hoặc aggregate dùng window khác. Nếu có thể, dùng chung logic transformation và so sánh feature giữa batch với online trên cùng ví dụ.

## 4. Feature store (mô hình khái niệm)

Feature store quản lý định nghĩa và giá trị feature để tái sử dụng. **Offline store** phục vụ training/phân tích lịch sử; **online store** phục vụ inference độ trễ thấp. Hai phía phải thể hiện định nghĩa và timestamp tương thích. Point-in-time correctness rất quan trọng: ví dụ lịch sử chỉ được dùng feature đã sẵn có tại thời điểm dự đoán.

Feature store là công cụ, không chữa được semantics kém. Trước tiên hãy định nghĩa entity key, feature timestamp, freshness, cách xử lý null, owner và access policy.

## 5. Version dữ liệu và snapshot

Version hóa raw input hoặc snapshot bất biến để có thể tái tạo thí nghiệm. Ghi schema, source, code, label và version transformation. Nếu dữ liệu quá lớn để sao chép, dùng content hash, partition manifest hoặc storage hỗ trợ snapshot, rồi ghi rõ thời gian lưu.

## Thực hành

Với fraud model, định nghĩa một feature chi tiêu trượt. Chỉ rõ event-time window, cách xử lý event muộn, transaction hiện tại có được tính không, training tái dựng feature tại từng timestamp lịch sử ra sao và online inference đọc giá trị mới thế nào.

## Lỗi thường gặp

- Chọn streaming vì nghe hiện đại hơn dù batch mỗi giờ đã đủ.
- Bỏ qua thời điểm event đến và dùng thông tin tương lai trong lịch sử replay.
- Tính offline feature với quy tắc null/default khác online feature.
- Cho rằng online store luôn mới mà không giám sát freshness và lỗi.
