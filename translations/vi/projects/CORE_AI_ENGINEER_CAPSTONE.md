# Core AI Engineer Capstone: Từ Câu hỏi đến Prediction Đáng tin cậy

Capstone kết hợp Python, data validation, classical ML, evaluation và AI engineering. Phạm vi đủ nhỏ để tái lập trên laptop. Mục tiêu là workflow có thể bảo vệ bằng lý lẽ, không phải điểm leaderboard.

## Đề bài

Chọn tabular dataset có license mở và prediction target rõ. Ví dụ dự đoán sự kiện tương lai, category hoặc phép đo số. Trước khi tải, ghi:

- ai dùng prediction;
- prediction được tạo ở thời điểm nào;
- hành động nào có thể theo sau;
- false positive và false negative nghĩa là gì;
- license data nào cho phép mục đích sử dụng dự kiến.

Không dùng dữ liệu cá nhân nhạy cảm. Nếu tác vụ ảnh hưởng con người, thêm kế hoạch human review và phân tích lỗi theo subgroup.

## Deliverable

Nộp repository gồm:

1. **Problem card:** target, thời điểm prediction, unit of analysis, intended use và out-of-scope use.
2. **Data card:** source, license, row grain, date range, thiếu hụt và bias đã biết.
3. **Data audit:** schema, missingness, giá trị sai, duplicate check và target prevalence.
4. **Split contract:** quy tắc group/time/stratification và leakage analysis.
5. **Baseline report:** rule đơn giản/constant predictor và metric định trước.
6. **Model comparison:** một baseline dễ diễn giải và ít nhất hai model family.
7. **Error report:** confusion matrix hoặc residual analysis, slice quan trọng và ví dụ lỗi.
8. **Run tái lập:** configuration, dependency, random seed cố định khi phù hợp, model đã lưu và hướng dẫn chạy.
9. **Prediction interface:** CLI hoặc API có schema validation và error rõ.
10. **Operations note:** latency, memory, cost kỳ vọng, monitoring signal, privacy/security risk và rollback condition.

## Giai đoạn 1 — Định hình tác vụ

Viết prediction contract trước khi xem score:

> Tại [thời điểm dự đoán], với mỗi [đơn vị], chỉ dùng [thông tin có sẵn] để ước lượng [target trong khoảng thời gian]. Output hỗ trợ [quyết định], nhưng không được dùng cho [quyết định loại trừ].

Định nghĩa population và unit. Nếu một người đóng góp nhiều row, nêu mục tiêu là dự đoán row mới từ người đã biết hay tổng quát hóa sang người chưa thấy.

**Exit check:** Reviewer biết được một row có nghĩa gì, feature tồn tại lúc nào và label được biết lúc nào.

## Giai đoạn 2 — Audit data trước cleaning

Tạo report gồm:

- số row và column;
- tính duy nhất của key và tỷ lệ duplicate;
- type, missingness và invalid-value count;
- target prevalence hoặc phân phối numeric target;
- phạm vi thời gian và tần suất group;
- leakage label có thể có và field chỉ xuất hiện sau outcome.

Giữ bản raw không sửa. Mọi cleaning rule phải tái lập và được biện minh bằng data contract.

**Exit check:** Giải thích được record nào bị loại, vì sao và việc loại có làm đổi target population không.

## Giai đoạn 3 — Cố định evaluation design

Chọn split dựa trên tuyên bố deployment:

- random và stratified cho ví dụ độc lập, exchangeable;
- tách group khi cần tổng quát hóa sang entity mới;
- theo thời gian khi dự đoán tương lai;
- kết hợp khi cả group và thời gian cùng quan trọng.

Chỉ fit imputation, scaling, feature selection và learned encoding bằng training data. Dùng validation để chọn model; giữ test cho workflow đã đóng băng.

**Exit check:** Train/validation/test không chia sẻ group bị cấm, thông tin tương lai hoặc duplicate record.

## Giai đoạn 4 — Xây baseline trước khi tăng độ phức tạp

So sánh tối thiểu:

1. constant baseline hoặc domain rule;
2. regularized linear/logistic model;
3. tree-based model như random forest hoặc gradient-boosted tree.

Với regression, dùng metric absolute/squared-error có ý nghĩa và kiểm tra residual. Với classification, báo class count, confusion matrix, precision/recall hoặc cost theo tác vụ, probability quality nếu quyết định dựa trên xác suất.

Không chọn model từ training score. Lưu run table có configuration, seed, training score, validation score và ghi chú.

## Giai đoạn 5 — Phân tích lỗi, không chỉ số trung bình

Kiểm tra ví dụ có:

- residual lớn nhất hoặc lỗi confidence cao;
- input thiếu/bất thường;
- group hoặc time window quan trọng;
- feature range phổ biến;
- prediction confidence thấp;
- false positive/negative tác động cao.

Mô tả vấn đề giống data quality, target mơ hồ, distribution shift, model capacity hay operational constraint. Không khẳng định nhân quả từ feature importance.

## Giai đoạn 6 — Package và serve

Chuyển data/prediction code có thể tái sử dụng ra khỏi notebook. Định nghĩa input contract có type, range cho phép, missing-value behavior và version. Xác minh model đã lưu cùng preprocessing được load đúng cặp.

Interface cần:

- từ chối input sai rõ ràng;
- trả prediction kèm model/version metadata;
- tránh log secret hoặc field nhạy cảm thô;
- xử lý an toàn lỗi model-load/dependency;
- có ví dụ tái lập nhỏ.

## Giai đoạn 7 — Monitor và rollback

Tách service health khỏi model quality.

**Service signal:** request count, error, timeout, latency percentile, memory và cost.

**Data signal:** schema change, missingness, feature range, population mix và delayed label.

**Quality signal:** task metric, calibration khi thích hợp, slice performance và action volume.

Nêu ai nhận alert, evidence nào kích hoạt điều tra và cách rollback về model trước hoặc safe fallback.

## Cấu trúc report gợi ý

1. Tóm tắt điều hành bằng ngôn ngữ dễ hiểu.
2. Problem/data contract.
3. Validation và leakage control.
4. Baseline và quy trình chọn model.
5. Kết quả kèm uncertainty và slice.
6. Ví dụ lỗi và giới hạn.
7. Hướng dẫn tái lập.
8. Kế hoạch deployment, monitoring, security và rollback.

## Rubric (100 điểm)

| Hạng mục | Điểm | Bằng chứng đạt tối đa |
| --- | ---: | --- |
| Định nghĩa bài toán và data license | 10 | Prediction unit/time, action, giới hạn và quyền sử dụng nêu rõ. |
| Data quality và provenance | 15 | Giữ raw data; issue và cleaning rule tái lập được. |
| Split và leakage control | 20 | Split khớp deployment; preprocessing học được fit bên trong train fold. |
| Baseline và so sánh model | 15 | Baseline có ý nghĩa; model được so theo cùng protocol. |
| Error analysis và uncertainty | 15 | Metric phù hợp; phân tích lỗi và slice subgroup/time. |
| Software và reproducibility | 10 | Learner khác cài, chạy, kiểm tra và tái lập kết quả. |
| Operations và responsible use | 15 | Interface, monitoring, privacy, security, cost và rollback được đề cập. |

## Câu hỏi review

- Điều gì làm score báo cáo trở nên vô hiệu?
- Nếu label definition thay đổi thì sao?
- Nhóm user nào có thể gặp error tệ hơn?
- Điều gì xảy ra nếu một input column biến mất?
- Làm sao biết model không còn hữu ích?
- Human có thể override output không; action đó có được ghi nhận không?

## Tiêu chuẩn hoàn thành

Capstone hoàn tất khi learner khác có thể tái lập baseline và model được chọn từ clean checkout, kiểm tra split, xem failure case đã biết và hiểu giới hạn deployment mà không cần hỏi bạn về lựa chọn chưa tài liệu hóa.
