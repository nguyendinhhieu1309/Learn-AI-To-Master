# Fine-tuning và Model Adaptation

## Mục tiêu học tập

Chọn giữa prompting, retrieval và fine-tuning; chuẩn bị dataset hợp lệ; đánh giá adaptation mà không làm nhiễm test set.

## 1. Chọn phương pháp theo vấn đề

- **Prompting** thay đổi instruction hoặc ví dụ của tác vụ nhưng không đổi weight model. Cách này nhanh để lặp và hữu ích cho formatting hoặc hành vi tác vụ đơn giản.
- **Retrieval** đưa kiến thức mới, riêng tư hoặc cần kiểm toán vào lúc inference. Có thể citation và cập nhật mà không huấn luyện lại.
- **Fine-tuning** điều chỉnh parameter bằng ví dụ tác vụ. Có thể cải thiện style, pattern tác vụ hoặc cách phản hồi chuyên ngành; không đảm bảo biến model thành cơ sở dữ liệu sự kiện hay luôn nhớ đúng dữ kiện đã học.

Có thể kết hợp các cách này. Hãy lập baseline và xác định nguồn lỗi: thiếu kiến thức gợi ý retrieval; định dạng tác vụ không ổn định gợi ý structured output/prompt; thiếu hụt hành vi lặp lại có thể cần fine-tuning.

## 2. Chất lượng training data

Dùng cặp input-output đại diện, nhãn rõ và được phép dùng để huấn luyện. Loại duplicate giữa train/validation/test. Chia theo nguồn, user hoặc thời gian khi ví dụ có liên quan. Thêm edge case và refusal mong muốn. Giữ evaluation set riêng, không liên tục dùng để lựa ví dụ hay hyperparameter.

Với supervised fine-tuning (SFT), ví dụ dạy model phản hồi mục tiêu. Preference optimization dùng so sánh hoặc sở thích để điều chỉnh output; cần dữ liệu preference đáng tin cậy và evaluation riêng. Nhãn kém hoặc dữ liệu lệch một phía làm hành vi kém hữu ích/an toàn.

## 3. Phương pháp parameter-efficient

Full fine-tuning cập nhật nhiều weight và cần đáng kể compute/storage. Parameter-efficient fine-tuning (PEFT), chẳng hạn LoRA, học một tập nhỏ parameter bổ sung (thường là cập nhật low-rank) trong khi phần lớn base weight được giữ cố định. Nhu cầu tài nguyên giảm nhưng vẫn cần dữ liệu tốt, versioning và evaluation. Quantization có thể giảm inference memory với một phần đánh đổi chất lượng.

## 4. Đánh giá adaptation

So model đã thích ứng với model gốc trên cùng task suite giữ lại. Đo task success, tuân thủ format, factuality, refusal behavior, regression trên ca tổng quát, latency và tài nguyên. Kiểm tra ghi nhớ và leakage, đặc biệt khi dữ liệu nhạy cảm. Version hóa base model, training dataset, hyperparameter, adapter và serving configuration.

## Thực hành

Giả sử assistant trả lời policy lỗi thời. Quyết định nên dùng prompt refinement, RAG hay fine-tuning. Câu trả lời tốt thường chọn retrieval cho policy thay đổi thường xuyên, cần citation nguồn; nêu thêm trường hợp fine-tuning vẫn có ích.

## Lỗi thường gặp

- Fine-tune để dạy dữ kiện thay đổi vốn nên được retrieval từ nguồn có thẩm quyền.
- Chỉ đo training loss hoặc một vài output được chọn đẹp.
- Để record gần trùng nhau lọt vào nhiều evaluation split.
- Quên rằng adapter, base model, tokenizer và prompt version phải được quản lý cùng nhau.
