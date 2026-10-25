# Agent Evaluation, Security và Operations

## Mục tiêu học tập

Tạo evaluation set cho hệ thống nhiều bước, kiểm tra trace và bảo vệ user/tool khỏi failure path phổ biến.

## 1. Đánh giá tác vụ, không đánh giá demo

Viết test case gồm yêu cầu bình thường, mơ hồ, thiếu thông tin, tool không khả dụng, tool output sai định dạng và input đối kháng. Định nghĩa outcome thành công trước khi xem câu trả lời model. Chạy lại cùng suite sau thay đổi prompt, model, tool hoặc retrieval.

Theo dõi task success, factual correctness, tool-call correctness, action không hợp lệ, refusal/abstention, cost, latency và khả năng phục hồi lỗi. Một điểm trung bình duy nhất che nhóm lỗi quan trọng; hãy báo category và severity.

## 2. Kiểm tra trace

Trace ghi model step, tool request/response, timing và final answer. Dùng trace xác định lỗi từ hiểu sai, retrieval, tool, xử lý state hay evaluation. Chỉ lưu dữ liệu cần thiết, che giá trị nhạy cảm, kiểm soát quyền và retention.

## 3. Nguy cơ và phòng vệ

- **Prompt injection:** instruction trong user message, trang web, tài liệu hoặc tool result tìm cách ghi đè system policy. Coi nội dung ngoài là data, cô lập rõ và thực thi permission ngoài model.
- **Excessive agency:** agent có thể thực hiện action rộng hoặc khó đảo ngược. Dùng least privilege, mặc định read-only, human confirmation rõ với action hệ trọng và thao tác có thể hoàn tác khi có thể.
- **Lộ dữ liệu:** prompt và trace có thể chứa thông tin cá nhân/bí mật. Thu thập tối thiểu, redact, hạn chế truy cập và đặt giới hạn lưu.
- **Tool misuse:** validate input, rate-limit, giới hạn phạm vi query và kiểm tra output. Không dựa vào model để enforce access control.
- **Hoàn tất bịa đặt:** yêu cầu bằng chứng tool thành công; phân biệt kết quả hoàn tất, một phần và thất bại.

## 4. Human review và escalation

Đặt ranh giới rõ lúc cần hỏi người: danh tính/permission không chắc, evidence mâu thuẫn, quyết định tác động cao hoặc action không thể đảo ngược. Human review cần là bước rõ trong workflow, không phải fallback khó hiểu sau lỗi.

## 5. Kiểm soát vận hành

Đặt timeout, max step, cost ceiling, monitoring alert, fallback và kill switch. Version hóa prompt, model, tool và knowledge index. Triển khai thay đổi theo giai đoạn và giữ đường rollback.

## Thực hành

Viết 12 evaluation case cho document assistant, gồm instruction được chèn vào tài liệu và search service không khả dụng. Với mỗi case, nêu hành vi tool mong đợi và phản hồi cuối an toàn. Review trace thay vì chỉ đọc văn bản cuối.

## Lỗi thường gặp

- Chỉ thử happy-path do chính tác giả hệ thống viết.
- Dùng LLM judge mà không có human review hoặc hiệu chỉnh.
- Tin delimiter tự nó ngăn prompt injection.
- Log mọi prompt và kết quả vô thời hạn vì có thể hữu ích sau này.
