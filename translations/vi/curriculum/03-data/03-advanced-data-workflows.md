# Workflow dữ liệu nâng cao

[Xem bài gốc bằng tiếng Anh](../../../../curriculum/03-data/03-advanced-data-workflows.md)

## Mục tiêu học tập

Chọn cách lưu trữ và xử lý phù hợp với dataset, truy vấn dữ liệu quan hệ an toàn, xử lý dữ liệu event/time series và duy trì khả năng nhìn thấy data lineage, chất lượng khi workflow phát triển.

## 1. SQL và dữ liệu quan hệ

Relational database lưu record trong table. **Primary key** xác định một hàng; **foreign key** liên kết hàng đó với bảng khác. Trước khi join, hãy hỏi một hàng đại diện cho điều gì ở mỗi phía. Nếu cả hai phía đều có key lặp, join có thể nhân số hàng và làm tổng bị phóng đại.

~~~sql
SELECT c.region, COUNT(DISTINCT o.order_id) AS orders,
       SUM(o.amount) AS revenue
FROM orders AS o
JOIN customers AS c ON o.customer_id = c.customer_id
WHERE o.created_at >= '2025-01-01'
GROUP BY c.region
ORDER BY revenue DESC;
~~~

WHERE lọc hàng trước aggregation; GROUP BY xác định group; HAVING lọc group sau aggregation. NULL nghĩa là không biết/vắng mặt, không phải số 0. So sánh như column = NULL không hoạt động như kỳ vọng; dùng IS NULL. Kiểm tra số hàng và tổng trước/sau join.

## 2. Chọn định dạng lưu trữ

- **CSV:** dễ kiểm tra và trao đổi; typing yếu, quét lặp trên file lớn kém hiệu quả.
- **JSON/JSONL:** record linh hoạt, hỗ trợ cấu trúc lồng nhau; thường lớn hơn và quét theo cột chậm hơn.
- **Parquet:** có kiểu, nén, dạng columnar; hữu ích cho phân tích chỉ đọc một số cột.
- **Database table:** hữu ích cho dữ liệu có cấu trúc dùng chung và truy vấn được kiểm soát.
- **Media file:** lưu image/audio/video riêng; lưu path, metadata, label và license trong index.

Giữ source thô có ghi chú và lưu output đã xử lý ở nơi khác. Tránh âm thầm ghi đè file nguồn.

## 3. Xử lý dữ liệu lớn hơn bộ nhớ

Trước tiên đo kích thước và xác định workload bị giới hạn bởi I/O, CPU hay memory. Chỉ chọn các cột cần thiết, filter sớm, dùng dtype hiệu quả, vectorize và xử lý theo chunk nếu có thể. Chia partition theo key/ngày hữu ích, nhưng tránh tạo quá nhiều file nhỏ khiến metadata chiếm phần lớn chi phí.

Distributed computing tăng thêm chi phí network, scheduling, serialization và consistency. Chỉ dùng khi cách làm trên một máy thực sự là bottleneck, không dùng mặc định. Trước khi mở rộng, kiểm tra liệu format columnar đơn giản hơn, đọc theo chunk hoặc aggregation bằng SQL có giải quyết được vấn đề không.

## 4. Time series và dữ liệu streaming

Với forecasting, dùng event time (thời điểm phép đo xảy ra), không chỉ processing time (thời điểm hệ thống nhận dữ liệu). Sắp xếp và kiểm tra timestamp, xử lý timezone và duplicate, định nghĩa rõ window. Feature trượt tại thời điểm t chỉ được dùng quan sát có sẵn trước mốc dự đoán; đưa giá trị tương lai vào sẽ gây leakage.

Trong event stream, dữ liệu có thể đến trễ, sai thứ tự hoặc lặp lại. Định nghĩa key khử trùng lặp, chính sách event trễ, thời lượng window và việc có cho phép sửa kết quả hay không. Theo dõi khoảng trống, clock drift và backpressure.

## 5. Nhu cầu dữ liệu chuyên biệt

- **Text:** giữ ngôn ngữ, nguồn tài liệu, timestamp và quyền truy cập; phát hiện lỗi encoding và tài liệu trùng/gần trùng.
- **Image/video:** giữ camera, resolution, timestamp, danh tính cảnh/người, phiên bản label và cách chia train/validation/test. Chia theo nguồn cảnh nếu các frame gần nhau có tương quan.
- **Sensor/robotics:** giữ đơn vị, frame ID, phiên bản calibration và timestamp đồng bộ. Không ghép phép đo từ frame khác nhau nếu chưa có transform.
- **Label:** theo dõi ai/cái gì tạo label, hướng dẫn annotation, confidence và các lần sửa.

## 6. Lineage, privacy và chất lượng theo thời gian

Ghi source/version dữ liệu, thời gian thu thập, license, transformation, schema và version đầu ra. Data card ngắn nên nêu cách thu thập, phần còn thiếu, bias đã biết, mục đích được phép dùng và giới hạn dự kiến. Giảm thiểu dữ liệu cá nhân; giới hạn quyền truy cập và thời gian lưu; không để secret trong log hay sample.

Mỗi lần pipeline chạy, hãy kiểm tra chất lượng dữ liệu: cột bắt buộc, type/range, tỷ lệ duplicate, missingness, category được phép, độ phủ thời gian và label balance. So sánh với mốc tham chiếu đã biết và điều tra thay đổi đáng kể. Không tự “sửa” thay đổi nếu chưa tìm nguyên nhân.

## Bài thực hành

Thiết kế pipeline ingestion cho đơn hàng hằng ngày. Ghi ý nghĩa mỗi hàng, key, schema, quy tắc duplicate, chính sách giá trị thiếu, timezone và validation. Giải thích cách phát hiện join vô tình làm doanh thu tăng gấp đôi. Với model dự đoán nhu cầu tương lai, liệt kê feature ứng viên có thể chứa thông tin sau cutoff dự báo.

## Lỗi thường gặp

- Join bảng mà không kiểm tra tính duy nhất của key và số hàng.
- Chuyển NULL thành 0 mà không biết 0 có ý nghĩa hay không.
- Chuyển sang distributed platform trước khi đo bottleneck.
- Xem cảnh báo chất lượng là lý do để tự động xóa/ghi đè record.
- Đánh mất provenance khi tạo dataset đã làm sạch hoặc export annotation.
