# Chất lượng dữ liệu, feature store và streaming

[Xem bài gốc bằng tiếng Anh](../../../../curriculum/03-data/09-data-quality-feature-stores-and-streaming.md)

AI đáng tin cậy bắt đầu từ dữ liệu đáng tin cậy. Pipeline phải giữ nguyên ý nghĩa, thời gian, lineage và chất lượng khi dữ liệu đi từ hệ thống nguồn tới training và inference.

## 1. Data contract và validation

Data contract mô tả schema, semantics, owner, freshness và kỳ vọng tương thích của table hoặc event stream. Cần validate nhiều hơn tên cột:

- Type, tỷ lệ null, miền giá trị, tính duy nhất và category được phép.
- Quan hệ key và event trùng.
- Timezone của timestamp, event time so với processing time và freshness.
- Định nghĩa label, label delay và tình trạng censoring đã biết.
- Thay đổi phân phối ở group và feature quan trọng.

Báo lỗi rõ khi field bắt buộc biến mất hoặc đổi ý nghĩa. Với thay đổi phân phối từ từ, hãy cảnh báo và điều tra thay vì âm thầm xóa hàng. Ghi lại quy tắc validation nào đã chạy và kết quả của chúng trong mỗi lần dựng dataset.

## 2. Pipeline idempotent và có thể khôi phục

Pipeline idempotent có thể chạy lại cùng input mà không tạo output trùng hoặc hỏng. Dùng key ổn định, transformation deterministic nếu có thể, ghi atomic, checkpoint và hành vi backfill tường minh. Tách raw input bất biến khỏi dataset curated để có thể tái tạo sau khi đổi logic.

Theo dõi lineage từ source partition qua transformation đến model artifact. Với mỗi training run, cần trả lời được: version dữ liệu nào, chính sách label nào, preprocessing code nào và cutoff thời gian nào đã tạo model này?

## 3. Event time, dữ liệu đến trễ và streaming

Streaming system nhận event bất đồng bộ. Event time là lúc event xảy ra; processing time là lúc hệ thống nhận biết nó. Event trễ/sai thứ tự cần chính sách: chờ watermark, sửa aggregate trước đó hoặc loại event quá cửa sổ trễ đã ghi.

Giao nhận end-to-end exactly-once giữa hệ thống độc lập rất khó. Thiết kế consumer idempotent, khử trùng bằng event ID và giám sát backlog cùng tỷ lệ event trễ. Feature real-time cần quy định freshness và cách xử lý khi không khả dụng.

## 4. Feature store và point-in-time correctness

Feature store có thể cung cấp định nghĩa feature nhất quán cho offline training và online serving. Thách thức chính là point-in-time correctness: hàng training tại thời điểm $t$ chỉ được dùng feature đã sẵn có tại hoặc trước $t$.

Join giá trị feature mới nhất vào hàng training lịch sử có thể leak thông tin tương lai. Dùng event timestamp và point-in-time join; kiểm tra bằng cách lấy mẫu hàng và lần ngược source record. Theo dõi owner, định nghĩa, đơn vị, giá trị mặc định, freshness và việc ngừng feature.

Offline/online skew có thể do code path, timezone, null default, aggregation window hoặc cache cũ khác nhau. Khi khả thi, chia sẻ logic transformation và chạy parity check trên cùng các record ví dụ.

## 5. Chất lượng dữ liệu và distribution drift

Vi phạm schema, feature thiếu, event trùng, dữ liệu cũ và lỗi label pipeline là vấn đề chất lượng. Thay đổi population có thể là dữ liệu mới hợp lệ và là tín hiệu drift. Hai trường hợp cần cách xử lý khác: sửa ingestion hỏng; điều tra drift hợp lệ cùng outcome label và bối cảnh sản phẩm.

Giám sát phân phối feature theo thời gian và cohort có ý nghĩa, nhưng không xem một drift score tổng quát là metric chất lượng model. Feature có thể drift mà không ảnh hưởng dự đoán; hoặc feature ổn định nhưng quan hệ với label lại thay đổi.

## 6. Thiết kế dataset cho machine learning

Chọn sampling frame phản ánh mục đích sử dụng. Lưu record ID ổn định và split assignment để các thí nghiệm so sánh cùng ví dụ. Ghi exclusion, deduplication, cách phân xử label và độ phủ subgroup. Kiểm soát quyền truy cập test data để tránh tuning lặp trên đó.

Khi label bị trễ, phân biệt dữ liệu gần đây chưa có nhãn với ví dụ negative. Censoring xảy ra khi outcome chưa có đủ thời gian xuất hiện; gán nhãn ngây thơ có thể làm lệch đánh giá có hệ thống.

## 7. Bài tập

1. Hàng training lịch sử được join với customer balance mới nhất, có thể đã cập nhật sau outcome target. Rủi ro là gì và cách xử lý?
2. Một event được xử lý hai lần sau retry. Pipeline cần làm gì để không nhân đôi aggregate?
3. Online feature mặc định bằng zero trong khi offline training điền missing bằng median. Có thể gây lỗi hệ thống nào?
4. Feature drift tăng nhưng quality trên traffic có label vẫn ổn định. Alert drift chứng minh điều gì và không chứng minh điều gì?

### Gợi ý đáp án

1. Có thể rò thông tin tương lai vào training. Dùng timestamp và point-in-time join theo lúc dự đoán.
2. Consumer phải idempotent, khử trùng bằng event ID hoặc ghi theo key với phép merge an toàn.
3. Training/serving skew: model được train và dùng inference với feature distribution, giá trị mặc định khác nhau.
4. Alert cho thấy phân phối đã đổi theo phép đo; nó không chứng minh accuracy giảm hay drift là nguyên nhân.
