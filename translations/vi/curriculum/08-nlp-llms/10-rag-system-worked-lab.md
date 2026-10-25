# Lab hệ thống RAG: Retrieval, Grounding và Evaluation

Retrieval-augmented generation (RAG) kết nối language model với nguồn kiến thức thay đổi hoặc riêng tư. Độ tin cậy phụ thuộc pipeline tài liệu, ranh giới retrieval, xử lý evidence và evaluation.

## Mục tiêu học tập

- Định nghĩa quy ước trả lời có căn cứ cùng hành vi abstention rõ ràng.
- Lập kế hoạch ingestion và retrieval giữ được provenance.
- Tách retrieval quality khỏi answer quality.
- Đánh giá citation, access control, prompt-injection resistance, latency và cost.

## Kiến thức tiên quyết

Hoàn thành [kỹ thuật retrieval](05-retrieval-engineering.md), [thiết kế prompt](04-prompt-design-and-structured-outputs.md) và [LLM evaluation](07-llm-evaluation-safety-and-cost.md). [Agent harness lab](../10-agentic-ai/08-agent-evaluation-harness-lab.md) bổ sung kiểm soát thực thi và trace cho hệ thống dùng tool.

## 1. Viết quy ước câu trả lời

Ví dụ:

> Trả lời câu hỏi về sổ tay kỹ thuật hiện hành chỉ bằng passages người dùng được phép đọc. Trích dẫn passages hỗ trợ claim factual. Nếu evidence thiếu, cũ hoặc mâu thuẫn, hãy nêu rõ và chỉ ra điều cần làm sáng tỏ.

Hãy xác định:

- collection nguồn nào thuộc phạm vi;
- cách xác định version tài liệu và quyền truy cập;
- citation nào được xem là đủ;
- khi nào hệ thống cần abstain;
- câu trả lời có thể tổng hợp nhiều tài liệu hay không;
- người dùng có thể làm gì khi không tìm được câu trả lời.

## 2. Giữ định danh nguồn xuyên suốt ingestion

Mỗi chunk được index phải giữ đủ metadata để định vị và kiểm soát nguồn:

- document ID ổn định và version;
- source URI hoặc đường dẫn repository;
- page, section hoặc character span;
- owner hoặc access-control label;
- ngày hiệu lực và lần cập nhật gần nhất;
- ingestion timestamp và parser version.

Không tạo chunk không thể truy về nguồn được phép. Nếu permission thay đổi, retrieval layer phải thực thi policy hiện hành; ẩn citation sau retrieval không phải access control.

## 3. Xem chunking là một retrieval experiment

Chunk size và overlap đánh đổi giữa độ đầy đủ context, retrieval precision và kích thước index.

- Chunk rất nhỏ có thể tìm được định nghĩa nhưng bỏ mất điều kiện.
- Chunk rất lớn làm loãng ranking và tiêu tốn context budget.
- Overlap giữ ngữ cảnh biên nhưng có thể tạo nhiều kết quả gần trùng.
- Heading, bảng, code và điều khoản pháp lý thường cần xử lý theo cấu trúc.

So sánh ít nhất hai cách chunking trên cùng query có nhãn. Ghi lại ranh giới chunk và giữ metadata tài liệu gắn liền. Đừng chọn phương án chỉ vì vài prompt tự viết cho kết quả đẹp.

## 4. Xây retrieval theo từng stage

1. **Query normalization:** bảo toàn ý định user; chỉ mở rộng abbreviation khi an toàn.
2. **Authorization filter:** thu hẹp không gian tìm kiếm trước khi lộ kết quả.
3. **Candidate retrieval:** so sánh lexical, dense-vector hoặc hybrid.
4. **Candidate ranking:** có thể rerank tập candidate nhỏ.
5. **Deduplication và diversity:** tránh context bị lấp bởi passage lặp lại.
6. **Context packing:** đưa evidence tốt nhất vào token budget cố định.
7. **Grounded generation:** yêu cầu claim bám evidence đã truy xuất.
8. **Citation validation:** xác nhận mọi nguồn được trích dẫn thuộc tập đã retrieve.

Đo riêng từng thành phần. Câu trả lời trôi chảy không thể phục hồi dữ kiện retrieval chưa cung cấp.

## 5. Đánh giá retrieval độc lập

Tạo query set có document ID hoặc span liên quan được gán nhãn. Đo:

$$
\operatorname{Recall@k}
=\frac{\text{số query có ít nhất một kết quả liên quan trong top }k}
{\text{số query}},
$$

cùng metric ranking như reciprocal rank hoặc normalized discounted cumulative gain nếu relevance có nhiều mức.

Bao gồm:

- cách diễn đạt chính xác và paraphrase;
- biến thể acronym và spelling;
- câu hỏi cần hai nguồn;
- query không có đáp án;
- tài liệu cũ và mâu thuẫn;
- query thuộc nhiều nhóm permission.

Metric retrieval phụ thuộc chất lượng annotation. Review mẫu “miss”: thiếu nhãn tài liệu liên quan có thể khiến kết quả tốt trông như xấu hoặc ngược lại.

## 6. Đánh giá riêng chất lượng câu trả lời

Với mỗi câu trả lời, đánh giá:

- correctness theo nguồn được phép;
- độ bao phủ các điểm bắt buộc;
- claim không có căn cứ hoặc bị nguồn phản bác;
- citation có tồn tại và có hỗ trợ claim không;
- abstention có phù hợp không;
- khả năng đọc và khả năng hành động.

Dùng rubric rõ ràng với ví dụ pass/fail. Automated evaluator giúp phân loại kết quả ban đầu, nhưng điểm số có sai số, không phải ground truth. Human review quan trọng với ứng dụng tác động cao.

## 7. Kiểm toán citation

Với mỗi câu factual:

1. Citation có trỏ tới nguồn đã retrieve không?
2. Span trích dẫn hỗ trợ toàn bộ claim hay chỉ một vế?
3. Version nguồn có còn hiện hành với câu hỏi không?
4. Có hai nguồn mâu thuẫn không?
5. Câu trả lời có dùng thông tin ngoài corpus được cấp quyền không?

Citation đúng cú pháp vẫn có thể không liên quan. Hãy theo dõi citation support, không chỉ citation presence.

## 8. Kiểm tra security và privacy

Coi văn bản retrieval là dữ liệu không tin cậy. Thêm ca tài liệu yêu cầu bỏ qua quy tắc trước, lộ secret hoặc gọi tool. Khi phù hợp, model cần tóm tắt nội dung ấy như dữ liệu, không làm theo chỉ thị.

Kiểm tra:

- retrieval chéo tenant và quyền tài liệu;
- prompt injection trong tài liệu và query parameter;
- nội dung ẩn/sai định dạng trong PDF và HTML;
- xóa nguồn và thu hồi permission;
- dữ liệu cá nhân trong log và trace;
- citation trỏ tới tài liệu user không được phép đọc.

Authorization thuộc retrieval boundary và cần được kiểm tra lại khi mở nguồn.

## 9. Latency, context và cost

Đo riêng từng stage:

$$
T_{\mathrm{request}}=
T_{\mathrm{query}}+
T_{\mathrm{retrieve}}+
T_{\mathrm{rerank}}+
T_{\mathrm{generate}}+
T_{\mathrm{validate}}.
$$

Theo dõi p50/p95/p99 latency, số token truy xuất, token model, retry và cost trên mỗi tác vụ trả lời thành công. Giảm retrieval $k$ có thể tiết kiệm nhưng giảm recall; context lớn có thể tăng latency mà không tăng evidence hữu ích.

## 10. Kế hoạch evaluation

Bắt đầu với 40–100 câu hỏi có nhãn nếu corpus và ngân sách cho phép. Gồm câu trả lời được, không trả lời được, multi-hop, nguồn cũ, mâu thuẫn và quyền truy cập. Giữ development set để lặp và held-out regression set để review cuối.

Với mỗi experiment, ghi:

- corpus snapshot và version parser/chunker;
- cấu hình embedding và ranking;
- prompt và model version;
- retrieval metric;
- điểm rubric câu trả lời và nhóm lỗi;
- latency và cost;
- giới hạn đã biết và evaluator version.

## Thực hành có đáp án kiểm tra

1. Retrieval recall thấp, nhưng câu trả lời chính xác khi nhận đúng context. Nên cải thiện gì trước?
2. Retrieval recall cao nhưng vẫn có claim không căn cứ. Cần kiểm tra stage nào?
3. Vì sao permission filtering cần diễn ra trước khi model thấy văn bản retrieval?
4. Recall@5 không cho biết điều gì?
5. Vì sao citation presence khác citation support?

### Đáp án

1. Cải thiện parsing, chunking, query handling, candidate retrieval hoặc ranking trước khi đổi generation.
2. Kiểm tra context packing, prompt constraint, mâu thuẫn nguồn, answer generation và citation validation.
3. Khi văn bản trái quyền đã tới model hoặc log, loại bỏ sau đó không thể đảo ngược việc lộ dữ liệu.
4. Nó không mô tả thứ tự xếp hạng trong top 5, độ đúng câu trả lời, citation, authorization hoặc latency.
5. Citation có thể tồn tại nhưng không suy ra được câu nó gắn kèm.

## Mini-project

Xây prototype RAG chỉ đọc trên tập tài liệu nhỏ được cấp phép. Gán nhãn ít nhất 40 câu hỏi với evidence span. So sánh hai cấu hình chunking hoặc retrieval, báo cáo retrieval metric riêng với answer metric, thêm ca không có đáp án và prompt injection, validate citation, ghi p95 latency và cost. Giữ lại held-out regression set.

## Lỗi thường gặp

- Quy lỗi cho generation khi retrieval đã bỏ sót.
- Tối ưu chunk size bằng vài prompt chọn sẵn.
- Coi citation do model tạo là evidence đã xác minh.
- Bỏ qua version tài liệu và metadata access control.
- Chỉ đánh giá query có đáp án.
- Dùng LLM judge mà không hiệu chỉnh hoặc spot-check bởi người.
