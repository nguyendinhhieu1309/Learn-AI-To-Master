# RAG, Evaluation và An toàn

## Mục tiêu học tập

Thiết kế hệ thống retrieval-augmented generation (RAG), tách chất lượng retrieval khỏi chất lượng câu trả lời và kiểm tra failure case trước khi phát hành.

## 1. Pipeline RAG

Một pipeline thường có hai pha:

1. **Index:** thu thập tài liệu được phép dùng, trích xuất văn bản, chia passage, gắn metadata nguồn, tạo embedding hoặc lexical index rồi lưu trữ.
2. **Answer:** diễn giải câu hỏi, truy xuất passage phù hợp, có thể rerank, đưa evidence được chọn vào context model và sinh câu trả lời có source reference.

Retrieval tự nó không đảm bảo câu trả lời có căn cứ. Model có thể bỏ qua evidence, đọc sai hoặc tự kết luận không có cơ sở.

~~~mermaid
flowchart LR
    Docs[Tài liệu được phép] --> Parse[Trích xuất và chia đoạn]
    Parse --> Index[Search index]
    Question[Câu hỏi user] --> Retrieve[Truy xuất passages]
    Index --> Retrieve
    Retrieve --> Evidence[Evidence được chọn]
    Evidence --> Generate[Sinh câu trả lời có nguồn]
    Generate --> Check[Kiểm tra căn cứ và citation]
    Check --> Response[Trả lời hoặc abstain]
~~~

## 2. Chunking và retrieval

Chunk rất nhỏ mất context; chunk quá lớn làm loãng relevance hoặc vượt context budget. Giữ title, số trang, ngày tháng và permission trong metadata. Kiểm tra kích thước chunk trên query thực tế. So sánh lexical search (tốt với từ khóa chính xác) và vector search (tốt với tương đồng ngữ nghĩa); hybrid có thể hữu ích. Chỉ truy xuất dữ liệu hệ thống được phép sử dụng.

## 3. Đánh giá từng giai đoạn

- **Retrieval:** recall@k, precision@k hoặc reciprocal rank trên câu hỏi có passage liên quan đã biết.
- **Answer:** correctness, completeness, groundedness, citation validity và khả năng abstain thích hợp.
- **System:** latency, cost, availability, privacy và hành vi trước abuse/input sai định dạng.

Tạo evaluation set nhỏ đã review, phản ánh kiểu câu hỏi thật và có câu không tìm thấy đáp án. Nếu có thể, tách nó khỏi tập tinh chỉnh prompt và retrieval. Automatic grader là tín hiệu tham khảo, không phải judge hoàn hảo; review thủ công một phần kết quả.

## 4. Citation và abstain

Yêu cầu model chỉ trả lời theo evidence truy xuất và chỉ ra nguồn cho từng claim factual. Validate passage được trích dẫn có tồn tại và đã thực sự được retrieve. Nếu evidence thiếu hoặc mâu thuẫn, hệ thống cần nói rõ hoặc hỏi lại thay vì đoán chắc chắn.

## 5. Security và privacy

Tài liệu truy xuất và câu hỏi user đều là input không tin cậy. Passage độc hại có thể chứa chỉ thị nhắm tới model (prompt injection). Đánh dấu ranh giới phần truy xuất, yêu cầu model coi nó là dữ liệu, hạn chế tools và không để nội dung không tin cậy cấp quyền. Xóa secret và dữ liệu cá nhân không cần thiết khỏi log. Kiểm soát quyền truy cập tài liệu trước retrieval, không phải sau khi sinh câu trả lời.

## Thực hành

Tạo 10 câu hỏi trên tập tài liệu nhỏ: trả lời được, mơ hồ, không trả lời được và đối kháng. Với mỗi câu, ghi evidence mong đợi và hành vi hệ thống cần có. Đánh giá retrieval độc lập với câu trả lời cuối. Có ít nhất một ví dụ mà câu tốt nhất là “Tôi chưa tìm thấy đủ bằng chứng”.

## Lỗi thường gặp

- Chỉ dùng ví dụ có vẻ hợp lý, không có câu hỏi không thể trả lời.
- Đo văn bản câu trả lời nhưng bỏ qua passage retrieve có liên quan không.
- Tin citation được sinh ra mà không kiểm tra nó có hỗ trợ claim không.
- Coi cách viết prompt là ranh giới bảo mật cho tool đặc quyền.
