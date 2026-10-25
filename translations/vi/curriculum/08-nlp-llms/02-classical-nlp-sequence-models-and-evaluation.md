# NLP cổ điển, Sequence Model và Evaluation

Language model hiện đại phát triển từ các ý tưởng NLP cổ điển. Hiểu tokenization, count-based model, sequence labeling và evaluation giúp chẩn đoán lỗi, chọn đúng công cụ.

## 1. Văn bản không phải chuỗi từ trung tính

Lựa chọn tokenization ảnh hưởng kích thước vocabulary, từ hiếm, hình thái học, dấu câu và ngôn ngữ không dùng khoảng trắng. Word tokenizer, character tokenizer và subword tokenizer tạo ra các đầu vào khác nhau. Chỉ normalization trong phạm vi tác vụ cho phép: lowercasing có thể bỏ tín hiệu chữ hoa; stemming quá mạnh có thể xóa mất nghĩa.

Với mỗi text dataset, ghi lại encoding, ngôn ngữ, normalization, truncation, cách xử lý văn bản thiếu và quy tắc gán nhãn. Chia train/test theo tác giả, hội thoại, nguồn tài liệu hoặc thời gian nếu các yếu tố đó có nguy cơ rò rỉ qua hai tập.

## 2. Bag-of-words và TF-IDF

Bag-of-words biểu diễn tài liệu bằng số lần token xuất hiện và bỏ qua thứ tự. TF-IDF giảm trọng số từ xuất hiện phổ biến trên nhiều tài liệu, đồng thời nhấn mạnh từ có tính phân biệt cho một tài liệu. Đây là baseline hiệu quả cho classification và retrieval.

Chỉ fit vocabulary và thống kê inverse document frequency trên training data. Dùng n-gram khi cần nắm thứ tự từ cục bộ, kiểm soát kích thước vocabulary để quản lý memory và feature nhiễu hiếm. Linear model như logistic regression hoặc linear SVM thường hoạt động tốt với vector sparse này.

## 3. Language modeling

Language model ước lượng xác suất chuỗi theo chain rule:

$$
P(w_1,\ldots,w_T)=\prod_{t=1}^{T}P(w_t\mid w_1,\ldots,w_{t-1})
$$

n-gram model xấp xỉ context bằng $n-1$ token trước đó. Count thưa cần smoothing để chuỗi chưa gặp không có xác suất bằng 0. Neural language model học distributed representation và dùng được context dài hơn, nhưng vẫn phụ thuộc tokenization, dữ liệu huấn luyện và objective.

Perplexity là hàm mũ của negative log-likelihood trung bình:

$$
\operatorname{PPL} = \exp\left(-\frac{1}{T}\sum_{t=1}^{T}\log P(w_t\mid w_{<t})\right)
$$

Nó đo độ phù hợp dự đoán trên test set đã token hóa. Không thể so sánh trực tiếp perplexity giữa tokenizer, dataset hoặc quy ước context khác nhau.

## 4. Sequence labeling: trực giác HMM và CRF

Named entity recognition, part-of-speech tagging và chunking gán nhãn cho từng token. Hidden Markov Model (HMM) dùng các giả định đơn giản hóa về transition và emission. Conditional Random Field (CRF) mô hình hóa điểm số toàn chuỗi đã chuẩn hóa và có thể áp đặt phụ thuộc giữa nhãn lân cận, chẳng hạn quy tắc chuyển hợp lệ giữa các nhãn entity.

Transformer encoder hiện đại thường thay thế sequence model cổ điển, nhưng góc nhìn structured prediction vẫn hữu ích: các nhãn không phải lúc nào cũng là quyết định độc lập; ranh giới entity cần được giải mã nhất quán.

## 5. Evaluation cho NLP

- **Classification:** precision/recall/F1 theo từng class, confusion matrix và calibration khi dùng xác suất.
- **Named entity:** nói rõ chấm exact-match hay theo token; định nghĩa cách chuẩn hóa span.
- **Machine translation:** BLEU đo độ trùng n-gram, có thể bỏ lỡ tương đương ngữ nghĩa; kiểm tra thêm chất lượng do người chấm và nhóm lỗi.
- **Summarization:** ROUGE đo độ trùng và có thể thưởng cho cụm từ được chép lại; factuality và độ bao phủ cần metric riêng.
- **Retrieval:** recall@k, mean reciprocal rank hoặc nDCG tại ngưỡng vận hành.

Dùng human review trên mẫu đại diện. Annotation guideline và bước phân xử rất quan trọng vì nhiều cách diễn đạt hợp lệ có thể cùng truyền đạt một đáp án. Với generative system, chấm riêng factual correctness, instruction following, safety và usefulness thay vì gộp thành một con số khó giải thích.

## 6. Data leakage trong văn bản

Tài liệu gần trùng lặp, văn bản mẫu, tên tác giả, lượt hội thoại và câu trả lời tương lai có thể làm lộ nhãn. Tìm duplicate giữa các split và xem xét n-gram có trọng số cao. Model nhận diện nguồn hoặc template có thể đạt điểm cao trên random split nhưng thất bại với nguồn mới.

## Bài tập

1. Vì sao TF-IDF + linear classifier vẫn là baseline hữu ích dù đã có transformer?
2. Language model có perplexity thấp hơn sau khi đổi sang tokenizer tạo số token gấp đôi. Điều đó có chắc là model tốt hơn không?
3. Summarizer có ROUGE cao nhưng bịa ngày tháng. Cần evaluation bổ sung nào?
4. Sentiment data có nhiều review sản phẩm chép từ template. Bạn sẽ tạo split thực tế hơn như thế nào?
