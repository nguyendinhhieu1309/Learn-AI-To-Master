# Model Lifecycle, Testing và Monitoring

ML model là một hệ thống có version gồm data, transformation, code, parameter và runtime dependency. MLOps đáng tin giúp toàn bộ hệ thống có thể tái lập, kiểm thử, quan sát và đảo ngược.

## 1. Version hóa input và output của model

Theo dõi tối thiểu:

- source revision và dependency lockfile;
- dataset snapshot, schema, label, split definition và preprocessing;
- training configuration, seed, hardware và metric;
- model artifact, tokenizer/feature map, calibration và decision threshold;
- evaluation report, giới hạn đã biết, trạng thái phê duyệt và deployment version.

File model thiếu preprocessing và label mapping không phải model sẵn sàng deploy. Khi có thể, dùng content hash hoặc dataset version bất biến.

## 2. Các tầng test cho ML system

- **Unit test:** transform tất định, feature logic và postprocessing.
- **Data test:** schema, range, null, uniqueness và label validity.
- **Training test:** smoke run nhỏ, loss hữu hạn, gradient và artifact mong đợi được tạo.
- **Model test:** output shape, probability range, invariant và baseline metric tối thiểu.
- **Integration test:** request parsing, serialization, model loading và response contract.
- **Performance test:** latency, throughput, memory và startup time.
- **Robustness test:** giá trị thiếu, request sai định dạng và edge case đã biết.

Test cần bắt regression mà không mã hóa cả training set hoặc âm thầm chấp nhận label semantics đổi.

## 3. Model registry và promotion

Registry gắn artifact với metadata, evaluation và lifecycle state. Promotion gate cần enforce yêu cầu theo mức rủi ro: quality threshold, slice check, latency budget, security review và lineage tái lập. Giữ staging environment đại diện production và rollback path về version đã biết tốt.

Shadow deployment chạy candidate trên input thật nhưng không dùng output để thay đổi kết quả user. Canary deployment đưa candidate vào tỷ lệ traffic nhỏ, kèm guardrail và điều kiện rollback tự động. Các kỹ thuật này giảm blast radius nhưng không thay offline evaluation.

## 4. Production monitoring

Theo dõi service health (error, latency, saturation) và model behavior (input schema, feature distribution, prediction distribution, confidence, abstention và delayed label). Input drift là tín hiệu điều tra, không chứng minh performance giảm. Ngược lại, input ổn định cũng không đảm bảo label hoặc quan hệ tác vụ ổn định.

Khi label đến muộn, dùng proxy signal thận trọng. Định kỳ đánh giá production example đã gán nhãn theo cùng label policy; so sánh theo thời gian và segment quan trọng. Theo dõi data freshness/missingness vì pipeline lỗi có thể giống model drift.

## 5. Retraining và rollback

Retraining có thể khởi chạy theo lịch, có label mới, quality drop đã đo hoặc upstream data thay đổi. Trước khi tự động hóa, định nghĩa trigger, quy tắc đưa data vào, evaluation gate, approver, deployment plan và rollback condition. Model mới không được thay production chỉ vì training job hoàn tất.

## 6. Xử lý data có trách nhiệm

Giới hạn data lưu, bảo vệ field nhạy cảm, kiểm soát log access và định nghĩa deletion/retention. Mặc định không log prompt, image hoặc identifier thô nếu không cần. Hạn chế quyền model artifact/dataset theo độ nhạy của nội dung.

## Bài tập

1. Registry có file model nhưng thiếu preprocessing code và label mapping. Còn thiếu gì để deploy an toàn?
2. Drift alert bật nhưng delayed label cho thấy performance ổn định. Nên phản hồi thế nào?
3. Candidate tốt hơn về offline accuracy nhưng latency gấp bốn. Cần promotion check nào?
4. Error rate deployment mới chỉ tăng ở một API version. Monitoring có version giúp thu hẹp nguyên nhân thế nào?
