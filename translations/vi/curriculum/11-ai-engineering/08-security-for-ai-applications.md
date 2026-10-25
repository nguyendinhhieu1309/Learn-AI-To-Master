# Security cho AI Application

## Mục tiêu học tập

Threat-model AI application, áp dụng least privilege cho tool/data và kiểm tra lỗ hổng từ hành vi model cũng như tích hợp hệ thống.

## 1. Threat model hệ thống

Liệt kê asset (data, credential, model endpoint, user account), trust boundary, actor, entry point và abuse có thể xảy ra. Xem xét model, tool, retrieval source, API, log, worker/sandbox và deployment account cùng nhau. Xác định user độc hại hoặc tài liệu bị đầu độc có thể tác động phần nào.

## 2. Bảo vệ tool và action

Dùng server-side authorization và allowlist. Validate typed argument và resource ownership ngay trước execution. Tách read-only tool khỏi tool tạo side effect. Yêu cầu approval cho action tốn kém, khó đảo ngược, hướng ra ngoài hoặc tác động cao. LLM quyết định gọi function không phải authorization.

## 3. Bảo vệ data và credential

Dùng secret manager, xoay credential, credential tồn tại ngắn và permission tối thiểu. Không gửi dữ liệu mật cho model provider trừ khi policy và consent user cho phép. Redact log, mã hóa data lưu và định nghĩa retention/deletion. Bảo vệ vector store/cache bằng tenant-aware filtering.

## 4. Prompt injection và điều khiển gián tiếp

User input và nội dung retrieval có thể chứa instruction thù địch. Delimit nội dung, hướng dẫn model coi là data, nhưng enforce security bằng code: ngăn văn bản không tin cậy đổi permission, gọi tool tùy ý hoặc lộ secret. Thử direct/indirect injection và kiểm tra trace kết quả.

## 5. Supply chain và sandbox

Pin/scan dependency, xác minh model artifact, review skill/script và MCP tool server trước khi bật, cô lập code execution. Python function hoặc skill chạy shell command có quyền của runtime trừ khi bị giới hạn. Dùng filesystem/network/CPU/time limit trong sandbox.

## 6. Security testing

Thử truy cập data chéo tenant, payload lỗi, file quá lớn, prompt injection, lạm dụng tool argument, request replay, rate limit, lộ secret và denial-of-service pattern. Báo finding theo khả năng khai thác và tác động. Tách dữ liệu red-team khỏi production data.

## Thực hành

Threat-model RAG agent có read tool search_docs và action send_email. Mô tả tấn công qua tài liệu retrieval độc hại, sau đó định nghĩa code-level control, điều kiện human approval, logging/privacy policy và test chứng minh ranh giới bảo mật.

## Lỗi thường gặp

- Tin system prompt là access-control boundary.
- Cấp shell, SQL hoặc filesystem cho model khi function hẹp đã đủ.
- Chỉ review prompt, bỏ qua dependency và tool-server code.
- Coi sandbox an toàn mà không xem network, file và credential permission.
