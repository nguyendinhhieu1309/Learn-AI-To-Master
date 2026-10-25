# Đánh giá RAG và Observability

Retrieval-augmented generation (RAG) kết hợp search với language model. Evaluation cần xác định lỗi đến từ retrieval, chọn evidence, generation hay chính câu hỏi user.

## 1. Xác định tác vụ trước khi chọn metric

Tạo test set đại diện gồm câu hỏi, thuộc tính câu trả lời mong đợi, tài liệu nguồn và trường hợp được phép abstain. Bao gồm câu hỏi không trả lời được, mơ hồ, nhiều bước suy luận (multi-hop), tài liệu dài, nguồn lỗi thời và cách diễn đạt đối kháng. Giữ test set khóa riêng khỏi thử nghiệm prompt và chunking.

Xác định thế nào là câu trả lời đúng: factual correctness, groundedness theo evidence, completeness, citation validity và refusal phù hợp có thể là các khía cạnh riêng. Câu ngắn vẫn có thể đúng dù khác câu chữ so với đáp án tham chiếu.

## 2. Đánh giá retrieval độc lập

Với mỗi query, gán nhãn passage hoặc tài liệu liên quan. Metric hữu ích:

- **Recall@k:** evidence liên quan có xuất hiện trong top $k$ không.
- **Precision@k:** tỷ lệ item liên quan trong top $k$ kết quả.
- **Mean reciprocal rank:** vị trí của kết quả liên quan đầu tiên.
- **nDCG:** chất lượng xếp hạng khi relevance có nhiều mức.

Kiểm tra query chính xác, semantic và query cần metadata. Đo retrieval latency và độ mới của index. Answer score cao trên tập nhỏ không chứng minh retrieval tốt nếu generation model vốn đã nhớ đáp án.

## 3. Đánh giá câu trả lời được sinh

Chấm correctness, evidence support, completeness, citation precision, citation recall và abstention. Citation precision hỏi passage trích dẫn có hỗ trợ claim gắn với nó không; citation recall hỏi các claim quan trọng có citation đầy đủ không. Citation nằm trong retrieved context chưa chắc là evidence hợp lệ.

Dùng deterministic check nếu có thể: schema validity, trường bắt buộc, citation ID có trong context và nội dung cấm. Dùng người đánh giá khi cần phán đoán tinh tế; hiệu chỉnh model-based judge theo human label. Khi so sánh hệ thống, reviewer không nên biết model nào tạo câu trả lời.

## 4. Quy nguyên nhân lỗi

Phân loại lỗi tại giai đoạn hỏng đầu tiên:

1. Query interpretation hoặc rewriting làm đổi ý người hỏi.
2. Search không tìm được evidence liên quan.
3. Reranking bỏ sót hoặc xếp sai passage hữu ích.
4. Chunking tách sự kiện khỏi ngữ cảnh.
5. Prompt không yêu cầu evidence hoặc abstention.
6. Model bỏ qua, hiểu sai hoặc mâu thuẫn evidence.
7. Citation parser/render gắn evidence sai.

Phân loại này giúp chọn đúng hướng sửa. Tăng model size không khôi phục tài liệu bị thiếu; cải thiện recall có thể làm giảm precision hoặc tăng context noise.

## 5. Ablation và kiểm soát thay đổi

Khi chẩn đoán pipeline, mỗi lần đổi một thành phần lớn: embedding model, chunk size, overlap, metadata filter, reranker, context budget, prompt hoặc generation model. Giữ baseline và ghi snapshot dataset/index. So sánh đồng thời quality, latency và cost. Báo cáo metric theo slice: ngôn ngữ, tuổi tài liệu, nguồn và loại câu hỏi.

## 6. Observability khi vận hành

Ghi trace an toàn, tôn trọng privacy về query transformation, ID/score của tài liệu truy xuất, version prompt/model, token count, latency từng stage, tool call, output validation và feedback. Tránh lưu raw content nhạy cảm nếu không cần; che dữ liệu hoặc hạn chế quyền khi cần giữ lại để debug.

Theo dõi tỷ lệ không tìm thấy kết quả, lỗi citation validation, abstention, latency, cost và xu hướng feedback. Cần nhãn human review đến trễ để ước lượng chất lượng câu trả lời thật; click và thumbs-up là proxy có thiên lệch.

## 7. Vòng lặp evaluation thực tế

1. Tạo error taxonomy và baseline test set.
2. Đo riêng retrieval và answer quality.
3. Review lỗi đại diện, kể cả câu nhìn có vẻ đúng nhưng thiếu evidence.
4. Thực hiện thay đổi có mục tiêu rồi chạy locked regression set.
5. Bổ sung ví dụ cho failure category chưa được bao phủ.
6. Canary deploy và theo dõi proxy chất lượng, tình trạng dịch vụ, cost, feedback.
7. Lấy mẫu production trace để người đánh giá xem xét theo quy trình privacy.

## Bài tập

1. Câu trả lời sai và đoạn đúng không có trong top-20 kết quả. Nên kiểm tra subsystem nào trước?
2. Retrieval recall tăng nhưng answer correctness giảm. Nêu hai giải thích có thể.
3. Citation trỏ tới tài liệu đã retrieve nhưng passage mâu thuẫn với claim. Metric citation nào phát hiện vấn đề?
4. Vì sao LLM judge có thể tỏ ra tự tin sai khi đánh giá output từ cùng họ model?
