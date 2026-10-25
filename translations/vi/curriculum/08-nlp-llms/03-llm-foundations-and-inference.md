# Nền tảng LLM và Inference

## Mục tiêu học tập

Giải thích luồng từ văn bản tới token rồi tới đầu ra model, xác định lựa chọn thiết kế nào ảnh hưởng latency/chi phí và chọn model phù hợp cho nhiệm vụ.

## 1. LLM dự đoán điều gì?

Autoregressive language model ước lượng phân phối của token tiếp theo dựa trên các token trước đó. Quá trình generation liên tục chọn token rồi nối vào context. Prompt có thể gồm instruction, lịch sử hội thoại, evidence truy xuất và kết quả tool; tất cả tiêu tốn context capacity.

Tokenizer chia đầu vào thành các đơn vị riêng của từng model. Số token không bằng số từ: code, số, thuật ngữ hiếm và một số ngôn ngữ có thể dùng nhiều token hơn. Token ID là chỉ mục biểu diễn, không phải điểm ngữ nghĩa.

## 2. Context window và attention

Context window giới hạn số token model xử lý trong một request (đôi khi output capacity được tính riêng). Context lớn chứa được nhiều evidence hơn nhưng tăng latency/chi phí và không đảm bảo model sử dụng chính xác mọi chi tiết. Context liên quan, có tổ chức thường hiệu quả hơn việc đưa vào mọi thứ.

Transformer dùng attention để kết hợp thông tin giữa các vị trí token. Khi autoregressive inference, key/value cache tránh tính lại biểu diễn token trước đó trong lúc sinh, đổi memory lấy tốc độ. Batching cải thiện mức sử dụng accelerator nhưng có thể làm tăng thời gian chờ của từng request.

## 3. Decoding controls

Greedy decoding chọn token kế tiếp có xác suất cao nhất. Temperature điều chỉnh lại logits trước khi lấy mẫu; top-p giới hạn việc lấy mẫu vào tập token đạt tổng xác suất mục tiêu. Các tham số này ảnh hưởng độ đa dạng và tính lặp lại, không kiểm chứng sự thật. Có thể dùng setting xác định để định dạng ổn định, sau đó vẫn cần validate đầu ra riêng.

## 4. Chọn model là chọn cả hệ thống

So sánh model trên bộ tác vụ của bạn, không chỉ benchmark tổng quát. Cân nhắc chất lượng, context, ngôn ngữ, modality, độ tin cậy khi gọi tool, latency, throughput, privacy, cost, vùng triển khai và khả năng cung cấp dịch vụ. Model nhỏ cộng retrieval tốt có thể vượt model lớn với tác vụ hẹp cần căn cứ.

Với model tự host, cân nhắc accelerator memory, chất lượng quantization, model license, batching và serving stack. Với model hosted, tính đến giới hạn request, điều khoản xử lý dữ liệu, thay đổi phiên bản và sự cố provider.

## 5. Ước lượng chi phí request

Ở mức tổng quát, chi phí phụ thuộc input token, output token, giá/compute của model và số lần gọi. Agent loop có thể thực hiện nhiều model call và tool call cho một yêu cầu. Đo chi phí toàn workflow trên mỗi tác vụ thành công, tính cả retry và retrieval, không chỉ lần gọi model đầu tiên.

## Thực hành

Lấy tài liệu policy dài và bộ câu hỏi thực tế. So sánh (a) gửi toàn bộ tài liệu với (b) truy xuất vài đoạn liên quan. Đo mức độ câu trả lời có căn cứ, input token và latency. Giải thích tình huống cách long-context hoặc retrieval thất bại.

## Lỗi thường gặp

- Nghĩ rằng context window lớn đảm bảo model chú ý mọi dữ kiện được đưa vào.
- Coi temperature là setting về độ thật hoặc an toàn.
- Chọn model từ benchmark không đại diện người dùng hay ràng buộc thực tế.
- Chỉ đo latency của model call, bỏ qua retrieval, tool và serialization.
