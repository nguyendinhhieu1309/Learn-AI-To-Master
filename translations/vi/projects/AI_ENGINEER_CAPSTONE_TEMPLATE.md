# Template AI Engineer Capstone

Copy file này vào project folder và điền trước khi triển khai. Project tốt chứng minh toàn bộ con đường engineering, không chỉ điểm model.

## 1. Problem và user

- Mô tả vấn đề:
- User dự kiến và người chịu ảnh hưởng:
- Prediction/action và thời điểm cần:
- Chi phí nếu kết quả sai:
- Trường hợp hệ thống không nên dùng:

## 2. Yêu cầu và success criteria

- Hành vi chức năng:
- Quality target và metric:
- Latency/throughput target:
- Cost/resource budget:
- Privacy, security, fairness và accessibility constraint:
- Human review/appeal path:

## 3. Data và governance

- Source, license, permission và version:
- Ý nghĩa row/example và schema:
- Missingness, duplicate, bias và giới hạn đã biết:
- Split strategy và leakage check:
- Data retention/deletion:

## 4. Baseline và experiment

- Baseline đơn giản nhất:
- Phương pháp candidate:
- Kế hoạch validation:
- Nơi lưu experiment log:
- Error category và subgroup analysis:
- Kết quả test cuối chưa đụng tới cùng uncertainty:

## 5. System architecture

~~~mermaid
flowchart LR
    Source[Data hoặc yêu cầu user] --> Validate[Validation và permission]
    Validate --> Prepare[Preprocess / retrieve context]
    Prepare --> Model[Model hoặc agent workflow]
    Model --> Verify[Output và policy check]
    Verify --> Product[API, UI, report hoặc robot interface]
    Model --> Observe[Log, metric, trace, evaluation]
~~~

- Component và owner:
- Version model/data/preprocessing:
- API/tool contract:
- Thiết kế state và checkpoint nếu cần:
- External dependency và fallback:

## 6. Reliability, security và operations

- Authentication và authorization:
- Secret và data access:
- Timeout, retry, idempotency và rate limit:
- Sandboxing và permission:
- Monitoring signal và alert owner:
- Incident response và rollback:
- Cost trên mỗi task thành công:

## 7. Tái lập và bàn giao

- Setup environment:
- Hướng dẫn data/model artifact:
- Lệnh run/evaluation chính xác:
- Output mong đợi:
- Failure mode đã biết:
- Việc tiếp theo:

## 8. Acceptance review

- [ ] Người khác tái lập được kết quả báo cáo.
- [ ] Model vượt hoặc có lý do thuyết phục so với baseline.
- [ ] Data split/evaluation khớp intended use.
- [ ] Error và limitation được tài liệu hóa.
- [ ] API/agent không thể vượt quyền được cấp.
- [ ] Monitoring vận hành và rollback được định nghĩa.
- [ ] Người chịu ảnh hưởng biết khi nào tin, nghi ngờ hoặc escalation kết quả.
