# Data engineering và validation: Bộ bài tập có lời giải

[Xem bài gốc bằng tiếng Anh](../../../../curriculum/03-data/PROBLEM_SET_WITH_SOLUTIONS.md)

Hãy viết các giả định trước khi đọc lời giải. Ví dụ nhỏ làm rõ cách suy luận; hệ thống production còn cần contract, monitoring và access control.

## 1. Cardinality của join

Table orders có 100 hàng, mỗi hàng là một order. Table line_items có 240 hàng. Join theo order_id thì inner join nên có bao nhiêu hàng? Điều gì có thể khiến số hàng vượt 240?

### Lời giải

Tối đa 240 nếu mỗi line item khớp đúng một order; ít hơn nếu có item không khớp. Nhiều hơn 240 nghĩa là order_id bị lặp trong orders hoặc điều kiện join chưa đầy đủ. Kiểm tra tính duy nhất của key và grain trước khi join.

## 2. Aggregate sau join one-to-many

Order A trị giá 30 và có ba line item. Order B trị giá 20 và có một item. Nếu cộng giá trị order sau khi join item với order thì kết quả ra sao?

### Lời giải

Tổng sau join là 30+30+30+20=110 vì order A bị lặp. Tổng đúng là 50. Aggregate tại grain của order trước khi join hoặc cộng amount thực sự ở grain line.

## 3. Point-in-time correctness

Lúc 09:00, model dự đoán default trong 30 ngày tới. Khoản trả nợ xảy ra lúc 10:00 và status nguồn đổi thành “đã trả” lúc 10:05. Feature cho prediction 09:00 có thể dùng status đó không?

### Lời giải

Không. Trạng thái chưa có sẵn khi đưa ra dự đoán. Theo dõi event time, source availability time và prediction time. Đảm bảo availability_time <= prediction_time, kể cả khi backfill về sau.

## 4. Chia theo entity

Dataset có 10 ảnh cho mỗi 100 patient. Vì sao random split theo ảnh có thể làm validation performance cao giả tạo?

### Lời giải

Ảnh cùng patient chia sẻ đặc điểm ngoại hình và cách thu thập. Split theo patient ID để một patient chỉ xuất hiện trong một partition. Thêm holdout theo thời gian hoặc site nếu deployment nhắm tới dữ liệu tương lai hay bệnh viện mới.

## 5. Fit preprocessing trên training data

Train có [1,3,5], validation có [100]. Bạn standardize bằng cả bốn giá trị. Sai ở đâu?

### Lời giải

Validation đã ảnh hưởng transformation được fit. Chỉ fit mean và standard deviation trên train; mean bằng 3 và population standard deviation bằng sqrt(8/3). Giữ nguyên các giá trị này cho validation, test và serving.

## 6. Missingness có thể mang thông tin

Giá trị cảm biến chỉ thiếu khi máy bị quá nhiệt. Missingness indicator có ích không? Chỉ median imputation có thể gây vấn đề gì?

### Lời giải

Bản thân việc thiếu có thể báo hiệu quá nhiệt nên indicator có thể giữ thông tin hữu ích. Xác minh tín hiệu có sẵn khi dự đoán và không phải do logging sau outcome. Chỉ median imputation có thể che khuất pattern đó.

## 7. Khử trùng lặp deterministic

Với mỗi entity, giữ event_time lớn nhất; nếu bằng nhau thì giữ ingestion_id lớn nhất. Mô tả cách làm deterministic.

### Lời giải

Partition theo entity, sắp xếp event_time giảm dần rồi ingestion_id giảm dần và giữ row number 1. Không dựa vào thứ tự file. Nếu dự kiến có retry, giữ bản ghi trùng để audit.

## 8. Event time và event đến muộn

Stream dùng event-time window 10 phút. Lúc processing time 12:20, hệ thống nhận event có event_time 12:02. Event thuộc window nào? Watermark làm gì?

### Lời giải

Event thuộc khoảng 12:00–12:10. Watermark ước lượng rằng event trước một mốc khó còn đến nữa để hệ thống chốt window. Chính sách lateness quyết định event trễ bị bỏ, sửa kết quả hay được đưa qua bước hiệu chỉnh.

## 9. Retry idempotent

Job hằng ngày thêm transaction vào bảng tích lũy, bị crash sau khi ghi rồi retry. Tránh tính lặp bằng cách nào?

### Lời giải

Dùng batch key ổn định cùng atomic upsert/merge, hoặc thay thế partition output bất biến khi retry. Cùng input phải tạo cùng kết quả đã commit. Append mù không idempotent.

## 10. Freshness của feature

Serving yêu cầu feature không cũ hơn 20 phút. Job đến trễ ghi giá trị dựa trên dữ liệu đã cập nhật từ 40 phút trước. Cần làm gì?

### Lời giải

Gắn timestamp cho feature và kiểm tra tuổi lúc sử dụng. Với dữ liệu cũ, theo chính sách tường minh: tính lại, dùng fallback an toàn, abstain hoặc trả lỗi có kiểm soát. Cảnh báo vi phạm freshness; không âm thầm coi giá trị cũ là mới.

## 11. Quality check

Age phải nằm trong [0,120]. Tuổi lớn nhất quan sát được là 87 và không có null. Điều đó chứng minh dữ liệu đúng chưa?

### Lời giải

Chưa. Giá trị vẫn có thể bị lặp, dịch chuyển, gắn nhãn sai, cũ hoặc sai ngữ nghĩa dù nằm trong miền. Bổ sung kiểm tra uniqueness, distribution, referential integrity, freshness và domain; xem sample cùng lineage.

## 12. Distribution shift

Population Stability Index (PSI) của feature là 0.32 sau khi ra mắt sản phẩm. Điều đó chứng minh accuracy của model giảm không?

### Lời giải

Không. PSI chỉ báo thay đổi phân phối theo cách chia bin đã chọn, không đo predictive quality hay quan hệ nhân quả. Kiểm tra thay đổi nguồn, lát cắt dữ liệu và ground truth đến trễ. Ngưỡng là heuristic, không phải luật phổ quát.

## 13. Xếp hạng bằng SQL

Với mỗi user, xếp event mới nhất trước. ROW_NUMBER và RANK khác nhau ra sao nếu hai event cùng timestamp mới nhất?

### Lời giải

ROW_NUMBER gán thứ hạng duy nhất; thêm tie-breaker deterministic nếu cần đúng một hàng. RANK gán cùng hạng cho các hàng hòa và bỏ trống hạng kế tiếp. DENSE_RANK cũng gán đồng hạng nhưng không bỏ số.

## 14. Định nghĩa grain của metric

Nêu hai ý nghĩa hợp lệ của “giá trị mua trung bình”.

### Lời giải

Trung bình mỗi transaction cho trọng số bằng nhau cho từng giao dịch. Trung bình mỗi customer tính average của từng người trước rồi cho mỗi customer trọng số bằng nhau. Cần nêu đơn vị, cách xử lý refund/cancel và time window.

## Checklist review

- Bạn có thể nêu grain và key của từng table không?
- Bạn có thể suy luận cardinality join và duplicate không?
- Bạn có thể chứng minh mọi feature có sẵn tại prediction time không?
- Split và preprocessing có theo đúng đơn vị, thời gian không?
- Freshness, retry, event trễ và correction có được định nghĩa rõ không?
- Quality check có kiểm tra ngữ nghĩa chứ không chỉ null/range không?
