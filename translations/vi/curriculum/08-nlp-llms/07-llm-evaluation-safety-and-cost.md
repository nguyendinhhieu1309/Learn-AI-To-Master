# Evaluation, An toàn và Chi phí LLM

## Mục tiêu học tập

Đánh giá LLM application như một hệ thống hoàn chỉnh; kết hợp review tự động và con người; quản lý chất lượng, rủi ro và chi phí vận hành.

## 1. Tạo evaluation set theo tác vụ

Soạn ví dụ từ người dùng mục tiêu và failure mode trước khi so model. Gồm trường hợp thông thường, edge case, mơ hồ, không thể trả lời, đa ngôn ngữ và đối kháng. Lưu kết quả mong đợi, cách trả lời chấp nhận được, yêu cầu evidence và mức độ nghiêm trọng. Tách ví dụ phát triển khỏi final holdout set.

## 2. Chấm nhiều khía cạnh

Tùy tác vụ, đo correctness, completeness, groundedness, citation support, schema validity, chất lượng refusal, độ chính xác tool-call, robustness, latency, cost và lỗi privacy. Báo cáo nhóm lỗi cùng ví dụ bên cạnh số trung bình. Model có điểm trung bình tốt vẫn có thể thất bại nghiêm trọng ở edge case quan trọng.

## 3. Human review và LLM judge

Human review hữu ích với mơ hồ, preference và lỗi tác động cao. Viết rubric, huấn luyện reviewer bằng ví dụ và đo mức đồng thuận nếu có thể. LLM judge mở rộng được bước đánh giá đầu nhưng có thể thiên vị câu dài, cùng điểm mù với generator hoặc nhạy với cách diễn đạt. Hiệu chỉnh judge với nhãn chuyên gia và tiếp tục kiểm tra mẫu bằng người.

## 4. Kiểm thử toàn hệ thống

Đánh giá extraction, retrieval, generation, tools, permissions, retries và trải nghiệm cuối. Thử instruction bị chèn trong nguồn, output sai định dạng, dependency không sẵn sàng, dự đoán confidence thấp và ranh giới quyền. Kiểm tra kết quả trong môi trường; transcript tuyên bố thành công không chứng minh hành động đã xảy ra.

## 5. Đánh đổi chất lượng và chi phí

Dùng model nhỏ/ít tốn kém nhất đáp ứng yêu cầu. Cân nhắc prompt caching, bỏ context không liên quan, retrieval, structured output, bounded retry và chuyển request dễ sang model nhỏ. Đo cost trên mỗi tác vụ thành công và tail latency, không chỉ giá mỗi token hay một call thành công.

## 6. An toàn và privacy

Giảm thiểu prompt được lưu, che trường nhạy cảm, giới hạn quyền đọc trace và đặt lịch xóa. Sàng lọc input không tin cậy khi cần, đồng thời kết hợp kiểm tra xác định, authorization và human oversight cho hành động hệ trọng.

## Thực hành

Tạo evaluation report cho document-answering assistant với 30 ca. Tách retrieval miss, claim thiếu căn cứ, citation sai, refusal không phù hợp và latency/cost. Đặt ngưỡng release cho nhóm lỗi nghiêm trọng và quy tắc chuyển sang người.

## Lỗi thường gặp

- Coi một điểm LLM-judge duy nhất là sự thật khách quan.
- Chỉ đánh giá văn bản sinh ra, bỏ qua permission, evidence truy xuất hoặc side effect thật.
- Tăng chất lượng trung bình nhưng để nhóm lỗi nghiêm trọng tăng.
- Ghi raw input nhạy cảm vì tiện debug.
