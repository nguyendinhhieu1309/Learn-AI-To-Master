# Kỹ thuật Retrieval

## Mục tiêu học tập

Xây pipeline retrieval trả về evidence hữu ích, chẩn đoán riêng lỗi retrieval và generation, đồng thời duy trì khả năng truy vết nguồn.

## 1. Pipeline indexing

1. Xác nhận quyền sử dụng nguồn; lưu provenance và version.
2. Trích xuất văn bản cùng metadata (title, author, date, section, page, ACL).
3. Chuẩn hóa cẩn thận; giữ heading, bảng, code và tín hiệu ngôn ngữ.
4. Chia tài liệu thành passage giữ được ý nghĩa và metadata.
5. Tạo embedding hoặc lexical index; ghi model/version embedding.
6. Lưu index và manifest có thể tái lập để dựng lại.

## 2. Chunking là bài toán đánh đổi

Chunk nhỏ tập trung nhưng có thể thiếu định nghĩa hoặc điều kiện liên quan. Chunk lớn giữ context nhưng có thể chôn câu cần tìm và chiếm context window. Nếu có thể, chia tại ranh giới ngữ nghĩa; chỉ overlap khi cần tránh mất ngữ cảnh ở biên. So sánh setting chunking trên query set đã được review thay vì chọn đại một token count.

## 3. Phương pháp retrieval

- **Lexical search** (BM25/inverted index): mạnh với cụm chính xác, identifier và thuật ngữ hiếm.
- **Dense/vector search:** ánh xạ query và passage thành embedding rồi xếp hạng theo similarity; khớp được paraphrase nhưng có thể bỏ lỡ identifier chính xác hoặc phủ định.
- **Hybrid retrieval:** kết hợp candidate lexical và semantic.
- **Reranking:** model mạnh hơn chấm relevance cho tập candidate nhỏ; có thể tăng chất lượng nhưng thêm cost/latency.

Áp dụng metadata filter và access control trước khi trả passage về. Lọc sau khi generation là quá muộn nếu văn bản trái quyền đã được đưa vào context model.

## 4. Đánh giá retrieval độc lập

Tạo câu hỏi có passage hỗ trợ đã biết. Đo recall@k (evidence liên quan có nằm trong top k không), metric precision/ranking và chất lượng theo loại tài liệu/ngôn ngữ. Kiểm tra trường hợp không có passage liên quan. Lỗi retrieval có thể đến từ extraction, chunking, query formulation, embedding, filter hoặc index lỗi thời.

## 5. Sinh câu trả lời có căn cứ

Chỉ đưa evidence liên quan cùng source ID và yêu cầu model trả lời dựa trên evidence. Yêu cầu citation truy ngược được tới span nguồn đã lưu. Kiểm tra citation bằng code và lấy mẫu để xem span có hỗ trợ claim không. Nếu evidence vắng mặt, mâu thuẫn hoặc cũ, hãy hỏi lại, nêu điều kiện hoặc abstain.

## 6. Bảo trì index

Xử lý document update/deletion, nguồn trùng, permission, embedding cũ và migration version. Ghi version index và embedding model trong mỗi lần chạy. Tôn trọng retention; xóa vector dẫn xuất khi nguồn gốc bắt buộc phải xóa.

## Thực hành

Tạo bộ evaluation 20 câu trên một nhóm tài liệu nhỏ. Bao gồm tra cứu chính xác, paraphrase, nhiều tài liệu, câu không trả lời được và câu bị giới hạn quyền. So sánh lexical, vector và hybrid retrieval. Gán stage lỗi cho từng trường hợp.

## Lỗi thường gặp

- Chỉnh generation prompt khi passage cần thiết chưa từng được retrieve.
- Bỏ qua source ACL lúc indexing hoặc query filtering.
- Coi vector similarity cao là bằng chứng hai passage cùng nghĩa.
- Để văn bản đã xóa hoặc cũ tồn tại trong embedding và cache.
