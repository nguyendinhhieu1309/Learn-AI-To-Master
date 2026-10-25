# Production Model Runbook: Serving, Monitoring và Rollback

Model đã train chưa phải production system. Hành vi đã deploy còn phụ thuộc preprocessing, input contract, model artifact, infrastructure, traffic, monitoring và quyết định vận hành.

## Mục tiêu học tập

- Đóng gói model cùng preprocessing và input/output contract.
- Chọn batch, online hoặc asynchronous serving theo yêu cầu sản phẩm.
- Lập budget latency/cost từ các thành phần đo được.
- Phân biệt service health với model quality.
- Định nghĩa rollout an toàn, incident response và rollback condition.

## Kiến thức tiên quyết

Hoàn thành [AI Engineering](README.md), nhất là [model serving](05-model-serving-and-inference-patterns.md), [reliability](07-reliability-observability-and-cost.md) và [inference latency/cost](09-inference-serving-latency-and-cost.md).

## 1. Đóng gói prediction contract

Release cần định danh nhiều hơn một file weight. Ghi:

- model artifact và checksum;
- preprocessing/feature schema version;
- training data snapshot hoặc reference;
- evaluation report và operating point dự kiến;
- dependency/runtime version;
- input range được hỗ trợ và cách xử lý missing value;
- owner, thời điểm tạo và rollback target.

Load model và preprocessing như một unit có version. Nếu model mới đòi feature order/normalization khác, deployment phải từ chối input không tương thích thay vì âm thầm trả score.

## 2. Chọn serving pattern

| Pattern | Phù hợp khi | Đánh đổi chính |
| --- | --- | --- |
| Batch | có thể tính trước prediction theo lịch | freshness bị giới hạn bởi khoảng batch |
| Online synchronous | request user cần phản hồi ngay | latency/availability nằm trên đường tới hạn |
| Asynchronous job | công việc tốn kém, có thể hoàn tất sau | cần job state, retry và thông báo hoàn tất |
| Edge/on-device | cần inference cục bộ do connectivity/privacy | memory, compute và update bị giới hạn |

Chọn theo freshness, latency, throughput, availability, privacy và cost; không dựa vào framework quen thuộc nhất.

## 3. Lập latency budget

Với synchronous request:

$$
T_{\mathrm{total}}=
T_{\mathrm{queue}}+
T_{\mathrm{network}}+
T_{\mathrm{validation}}+
T_{\mathrm{preprocess}}+
T_{\mathrm{model}}+
T_{\mathrm{postprocess}}.
$$

Đo phân phối end-to-end và từng component. Báo p50, p95, p99; mean có thể che tail failure. Đặt deadline và truyền tiếp để request đã timeout không tiếp tục tiêu thụ tài nguyên vô hạn.

Batching cải thiện accelerator utilization nhưng thêm queue delay. Đo latency dưới concurrency thực tế, không chỉ một request trong notebook.

## 4. Theo dõi ba lớp

### Service health

- request volume, error rate, timeout rate;
- p50/p95/p99 latency;
- CPU/GPU, memory, queue depth và saturation;
- cost mỗi request hoặc task thành công.

### Data health

- schema và feature availability;
- missingness, range và category frequency;
- input volume và freshness;
- thành phần nhóm và thời gian;
- feature parity giữa training và serving.

### Model/task quality

- metric từ delayed label;
- calibration và action rate nếu quyết định dựa xác suất;
- error slice theo population/operating condition quan trọng;
- drift indicator và human override rate.

Input drift đơn lẻ không chứng minh model quality giảm. Quality alert cần label hoặc proxy đã xác thực; giữ rõ khác biệt này.

## 5. Rollout theo giai đoạn kiểm soát

1. Validate artifact và input contract offline.
2. Chạy shadow deployment, ghi prediction nhưng không đổi outcome user.
3. So latency, error và input behavior với version hiện tại.
4. Nếu sản phẩm cho phép, chuyển một phần traffic nhỏ thành canary có monitor.
5. Chỉ tăng traffic khi health/quality gate định trước đều đạt.
6. Giữ version trước và safe fallback.

Với model ảnh hưởng quyết định, định nghĩa rollback criteria trước launch. Ví dụ error rate tăng liên tục, latency vượt service objective, feature contract hỏng hoặc quality regression đáng kể trên label đã đủ trưởng thành.

## 6. Incident checklist

Khi prediction có vẻ sai:

1. xác định lỗi thuộc input, preprocessing, model, serving hay product policy;
2. kiểm tra release và data-contract change gần đây;
3. lấy mẫu trace được phép đã redact;
4. so feature distribution và model version;
5. dừng hoặc chuyển traffic về baseline an toàn nếu vượt ngưỡng rủi ro;
6. giữ evidence cho root-cause analysis;
7. ghi correction, owner và prevention test.

Đừng tự retrain chỉ vì một drift detector báo động. Xác minh nguyên nhân và đánh giá label, business impact, safety trước.

## 7. Cost trên mỗi tác vụ thành công

Cost/request thô có thể ưu tiên hệ thống rẻ nhưng thường thất bại. Trên evaluation window:

$$
\text{cost trên mỗi tác vụ thành công}
=\frac{\text{tổng chi phí serving và dependency}}
{\text{số tác vụ đạt success contract}}.
$$

Báo denominator, định nghĩa failure, traffic mix và time window. Với generative system, tính retry, tool call, retrieval và evaluation cost khi đáng kể.

## Thực hành

1. Batch job chạy mỗi giờ, nhưng sản phẩm cần prediction trong 100 ms. Pattern nào phù hợp hơn và cần đo gì?
2. p50 latency 30 ms, p99 là 900 ms trong khi deadline 200 ms. Service có khỏe không?
3. Feature distribution dịch chuyển nhưng chưa có label. Có thể kết luận gì?
4. Model mới accuracy offline cao hơn nhưng 2% request lỗi do schema mismatch. Có nên tiếp tục rollout?
5. Vì sao cần giữ artifact trước cùng preprocessing version?

## Đáp án kiểm tra

1. Online synchronous là ứng viên; đo end-to-end tail latency thực tế, throughput, saturation, failure rate và cost.
2. Tail vi phạm deadline. Median/average không đáp ứng yêu cầu service ở từng request.
3. Có thể báo input drift và điều tra. Thiếu label/proxy hợp lệ thì chưa thể kết luận task quality giảm hay tăng.
4. Không. Schema-contract failure chặn release; sửa mismatch artifact/interface rồi validation lại.
5. Rollback cần khôi phục toàn bộ prediction behavior, không chỉ weight phụ thuộc transformation khác.

## Mini-project

Chọn model capstone và viết runbook nêu serving mode, request contract, latency budget, monitoring dashboard, release gate, incident owner, fallback, rollback trigger và trace ví dụ đã redact. Mô phỏng input schema thay đổi và chứng minh hệ thống từ chối hoặc xử lý an toàn.

## Lỗi thường gặp

- Monitor CPU/latency nhưng bỏ qua model quality.
- Gọi input drift là performance regression khi chưa có label.
- Benchmark một request thay vì concurrency thực.
- Chỉ rollback model weight mà quên preprocessing.
- Đặt alert không có owner hoặc hành động phản hồi.
