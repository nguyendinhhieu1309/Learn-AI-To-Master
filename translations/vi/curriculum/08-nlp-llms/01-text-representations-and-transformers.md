# Biểu diễn văn bản và Transformer

## Mục tiêu học tập

Giải thích văn bản được chuyển thành đầu vào của model như thế nào, mô tả ý tưởng chính của transformer và nhận biết giới hạn của đầu ra language model.

## 1. Văn bản là dữ liệu có cấu trúc

Trước hết hãy xác định nhiệm vụ: phân loại đánh giá sản phẩm, tìm câu trả lời trong tài liệu, dịch hay sinh văn bản. Bảo toàn cấu trúc có ích như dấu câu, ngôn ngữ, code block và ranh giới tài liệu. Chuẩn hóa (normalization) như chuyển chữ thường, bỏ markup, chuẩn hóa khoảng trắng có thể giúp một số tác vụ nhưng cũng làm mất thông tin. Viết hoa có thể phân biệt tên riêng; chữ hoa thường có thể quan trọng trong code.

## 2. Token và embedding

Tokenizer chia văn bản thành các đơn vị gọi là **token**. Token có thể là từ hoàn chỉnh, subword, ký tự hoặc byte. Token ID là số nguyên dùng làm chỉ mục, bản thân nó không mang ý nghĩa. Embedding table ánh xạ mỗi token ID thành vector để model xử lý. Tokenization ảnh hưởng context length, từ hiếm, văn bản đa ngôn ngữ và chi phí.

Các cách biểu diễn cổ điển gồm bag-of-words và TF-IDF. TF-IDF gán trọng số cao hơn cho thuật ngữ thường gặp trong một tài liệu nhưng ít phổ biến trên toàn bộ corpus. Linear model chạy trên TF-IDF là baseline rẻ và hữu ích cho nhiều bài toán phân loại văn bản.

## 3. Attention và Transformer

Self-attention cho phép mỗi token tổng hợp thông tin từ các token khác dựa trên độ tương đồng đã học giữa query và key. Transformer block kết hợp attention với feed-forward network, thường có residual connection và normalization. Positional encoding hoặc position embedding cung cấp thông tin thứ tự.

Causal language model che (mask) các token tương lai và dự đoán token kế tiếp. Encoder model có thể dùng ngữ cảnh cả hai chiều để phân loại hoặc tạo representation.

Pretraining học các quy luật thống kê tổng quát từ corpus lớn. Fine-tuning điều chỉnh model cho nhiệm vụ bằng ví dụ. Cả hai đều không đảm bảo kiến thức đúng sự thật hoặc công bằng. Dữ liệu huấn luyện, chất lượng nhãn, prompt và decoding đều ảnh hưởng kết quả.

## 4. Generation là dự đoán, không phải kiểm chứng

Autoregressive generation lặp lại việc chọn hoặc lấy mẫu token kế tiếp. Temperature và top-k/top-p thay đổi độ đa dạng đầu ra, không xác minh sự thật. Câu trả lời trôi chảy vẫn có thể sai, lỗi thời hoặc thiếu căn cứ. Với tác vụ factual, hãy đưa bằng chứng liên quan và xác minh kết luận có suy ra được từ bằng chứng đó hay không.

## 5. Chọn baseline và đo lường

Với text classification, hãy bắt đầu bằng TF-IDF + logistic regression. Với search, bắt đầu bằng lexical search. Chỉ so sánh neural model sau khi đã xác định metric và test set đại diện. Theo dõi ngôn ngữ, loại tài liệu và nhóm người dùng; metric tổng hợp có thể che giấu nhóm được hỗ trợ kém.

~~~python
from sklearn.feature_extraction.text import TfidfVectorizer
from sklearn.linear_model import LogisticRegression
from sklearn.pipeline import make_pipeline

texts = ["easy to use", "works very well", "not useful", "poor quality"]
labels = [1, 1, 0, 0]
baseline = make_pipeline(TfidfVectorizer(), LogisticRegression())
baseline.fit(texts, labels)
print(baseline.predict(["useful and easy"]))
~~~

Ví dụ nhỏ này minh họa pipeline, không phải evaluation có ý nghĩa: bốn mẫu quá ít để kết luận hiệu năng. Với bài toán thực, dùng split đại diện và báo cáo lỗi theo class, ngôn ngữ.

## Bài tập ngắn

Chọn câu có tên riêng hiếm, emoji và code identifier. So sánh cách hai tokenizer chia câu đó. Giải thích vì sao số lượng token khác với ý nghĩa token. Viết một khẳng định generative model có thể đưa ra về câu và nêu bằng chứng cần có để xác minh.

## Lỗi thường gặp

- Coi token ID là từ hoặc giá trị số của token ID là khoảng cách ngữ nghĩa.
- Bỏ dấu câu hoặc chữ hoa mà chưa kiểm tra tác vụ có phụ thuộc chúng không.
- Nghĩ rằng model sinh văn bản trôi chảy đã kiểm tra nguồn.
- Bỏ qua baseline đơn giản như retrieval hoặc TF-IDF.
