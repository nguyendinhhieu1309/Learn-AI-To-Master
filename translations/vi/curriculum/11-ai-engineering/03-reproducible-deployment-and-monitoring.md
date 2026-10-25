# Deployment Tái lập và Monitoring

## Mục tiêu học tập

Giữ nhất quán mối quan hệ giữa data, preprocessing, model và output; serve prediction ổn định; monitor hệ thống sau release.

## 1. Tái lập kết quả model

Ghi source data/version, split definition, code revision, dependency, configuration, random seed, hardware khi phù hợp và metric. Lưu preprocessing đã fit cùng estimator. Model serialize mà thiếu tokenizer, scaler, category mapping hoặc thứ tự label là chưa đầy đủ.

Reproduction nghĩa là người khác có thể theo bước đã tài liệu hóa và thu được kết quả tương đương. Khác biệt floating-point/accelerator có thể ngăn bit-for-bit identity; hãy nêu tolerance và variance.

## 2. Đóng gói inference

Định nghĩa input/output schema ổn định: field bắt buộc, type, unit, range cho phép và cách xử lý giá trị thiếu. Áp dụng đúng preprocessing như lúc train. Validate request, đặt giới hạn size/time và trả error rõ thay vì âm thầm đổi input sai định dạng.

Với batch job, throughput có thể quan trọng nhất. Với interactive API hoặc robotics, tail latency và availability có thể quan trọng hơn. Đo trên runtime và hardware mục tiêu.

## 3. Release an toàn

Dùng configuration riêng cho development, staging và production. Kiểm tra ví dụ đã biết, input sai và tính tương thích model/preprocessor trước release. Version hóa artifact và giữ rollback path. Shadow/canary deployment có thể phát hiện input thật khác dự kiến trước khi mở rộng, tùy privacy và điều kiện vận hành.

## 4. Monitor service và model

Service monitoring gồm error, latency percentile, throughput, resource use và availability. Data monitoring gồm thay đổi schema, tỷ lệ thiếu, feature range, tần suất category và input drift. Model monitoring gồm nhãn đến trễ, quality metric, calibration và hành vi theo nhóm khi phù hợp.

Input drift là tín hiệu cần điều tra, không phải bằng chứng performance giảm; phân phối input ổn định cũng không chứng minh model còn chính xác. Dùng label hoặc audit phù hợp để đo quality. Định nghĩa ai phản hồi và alert kích hoạt hành động gì.

## 5. Documentation và governance

Giữ model card nêu intended use, evaluation data, metric, subgroup result khi có, giới hạn và maintenance owner. Tôn trọng data license, privacy, retention, access control và yêu cầu tổ chức áp dụng. Với robotics/hệ tác động cao, ghi safe state và safety control độc lập.

## Thực hành

Đóng gói một model cùng preprocessing thành inference function/service nhỏ. Thử input hợp lệ, thiếu, ngoài range và category lạ. Ghi artifact version cùng rollback plan. Xác định metric quality chỉ theo dõi được sau khi label tới.

## Lỗi thường gặp

- Cập nhật preprocessing tách khỏi model.
- Báo động drift mà không có owner hoặc bước điều tra.
- Tự động retrain trên data mới mà không kiểm tra chất lượng data/label.
- Coi offline score cao là bằng chứng production an toàn.
