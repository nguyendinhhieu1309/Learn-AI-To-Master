# Container, Cloud và Deployment Architecture

## Mục tiêu học tập

Hiểu cách đóng gói AI application, xác định cloud resource cần thiết và lên kế hoạch release có thể quan sát/rollback.

## 1. Container

Container image đóng gói application code, runtime và dependency. Nó giúp development/deployment environment đồng nhất hơn. Image không tự bao gồm model weight, credential, GPU driver hay external service; cần định nghĩa cách mount/tải chúng và xác nhận tương thích.

Giữ image tái lập và vừa phải: chọn base image cụ thể, cài dependency đã pin, không copy secret/data cục bộ và chạy bằng non-root user nếu khả thi. Tách build-time configuration khỏi runtime secret.

## 2. Thành phần cloud

AI system thường dùng compute (CPU/GPU), object storage cho dataset/artifact, database/cache cho application state, queue cho asynchronous job, model endpoint và observability service. Chọn managed hoặc self-hosted theo năng lực vận hành, quy mô, latency, data residency, cost và yêu cầu tránh lock-in.

Dùng identity/access management để cấp từng service đúng quyền tối thiểu cần. Model-serving container không nên có quyền rộng tới storage/database không liên quan.

## 3. CI/CD và environment

Continuous integration có thể chạy formatting, static check, unit/contract test, security scan và một số model evaluation chọn lọc trên thay đổi đề xuất. Deployment pipeline promotion cùng một build artifact qua development, staging, production với configuration riêng theo environment. Model/data artifact có version và approval criteria riêng.

Dùng canary/shadow release khi rủi ro và kiến trúc cho phép. Ghi release version và cung cấp rollback. Infrastructure-as-code giúp cloud resource dễ review/tái lập.

## 4. Kubernetes và orchestration (khái niệm)

Container orchestration xếp lịch workload, khởi động lại process lỗi và scale replica. Kubernetes cung cấp primitive như pod, deployment, service, job. Nó thêm độ phức tạp vận hành: bảo mật cluster, networking, GPU scheduling, observability, nâng cấp và capacity. Học khái niệm nhưng không bắt buộc Kubernetes cho project một service nhỏ.

## 5. Accelerator và model artifact

Đảm bảo runtime library, device driver, model format và GPU capability tương thích. Model weight lớn có thể tải khi startup hoặc mount từ artifact storage; cân nhắc cold-start time, checksum/verification, license và cache policy. Model quantized giảm memory/cost nhưng cần evaluation lại về quality.

## Thực hành

Vẽ architecture cho private document assistant. Đánh dấu nơi lưu source file, vector index, model endpoint, API credential, conversation state và trace. Với mỗi component, định nghĩa access policy, backup/recovery kỳ vọng và version information.

## Lỗi thường gặp

- Đưa credential vào image layer hoặc container environment có thể lộ qua log.
- Coi container là bằng chứng deployment tái lập.
- Thêm orchestration trước khi hiểu workload và cách nhóm vận hành.
- Cập nhật model file tại chỗ mà không có checksum, version hoặc rollback.
