# AI Engineering

Module này kết hợp software engineering, MLOps, deployment và thiết kế AI system. Mục tiêu là làm hệ thống AI có thể tái lập, kiểm thử, triển khai, quan sát và bảo trì.

## Kết quả học tập

- Chuyển notebook và experiment thành Python project/API có thể bảo trì.
- Version hóa data, code, model và configuration để tái lập kết quả.
- Kiểm thử data và hành vi model; quản lý model release và rollback.
- Chọn serving pattern và phân tích latency, throughput, reliability, cost.
- Bảo vệ AI application và theo dõi cả service health lẫn task quality.

## Kiến thức tiên quyết

Python, Git, xử lý dữ liệu và một model ML/DL đã huấn luyện. HTTP/API cơ bản và dùng command line cũng hữu ích.

## Lộ trình gợi ý

### Nền tảng software

1. [Python production và cấu trúc project](01-production-python-and-project-structure.md)
2. [Git, testing, API và data contract](02-git-testing-apis-and-contracts.md)

### Model lifecycle và deployment

3. [Deployment tái lập và monitoring](03-reproducible-deployment-and-monitoring.md)
4. [Model lifecycle, testing và monitoring](04-model-lifecycle-testing-and-monitoring.md)
5. [Model serving và inference pattern](05-model-serving-and-inference-patterns.md)
6. [Container, cloud và deployment](06-containers-cloud-and-deployment.md)

### Vận hành production system

7. [Reliability, observability và cost](07-reliability-observability-and-cost.md)
8. [Bảo mật AI application](08-security-for-ai-applications.md)
9. [Inference serving, latency và cost](09-inference-serving-latency-and-cost.md)
10. [Production model runbook](10-production-model-runbook.md)
11. [LLMOps evaluation và experiment design](11-llmops-evaluation-and-experiment-design.md)

## Bài thực hành

- Refactor notebook thành Python package nhỏ có configuration, log và command tái lập.
- Tái tạo kết quả model trong clean environment; ghi data, dependency và model version.
- Đóng gói model sau request/response contract có validation; so batch với online inference.
- Mô phỏng outage dependency hoặc đổi input schema; xác minh timeout, fallback, monitoring và rollback.
- Benchmark traffic thực tế; báo p50/p95/p99 latency, throughput, memory và cost trên mỗi tác vụ thành công.
- Viết và diễn tập [production model runbook](10-production-model-runbook.md), gồm lỗi schema mô phỏng và rollback.
- Hoàn thành [LLMOps evaluation project](11-llmops-evaluation-and-experiment-design.md) với test set version hóa, so sánh paired, phân tích slice và release gate.

## Dự án checkpoint

Hoàn thiện một model application từ đầu tới cuối, kèm source/data lineage, input validation, evaluation evidence, cấu hình deployment, monitoring service và task, ghi chú bảo mật, ước lượng chi phí và rollback plan đã kiểm chứng.
