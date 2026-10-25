# Các Retrieval Pattern RAG Nâng cao

Retrieval nâng cao là lời giải cho một failure mode đã được chẩn đoán. Đừng thêm graph database, query rewriting hoặc reranker trước khi đo câu hỏi nào baseline bỏ sót và vì sao.

## Mục tiêu học tập

- Chọn cách chunking và indexing phù hợp cấu trúc tài liệu.
- Kết hợp lexical và semantic search bằng rank fusion.
- Giải thích query rewriting, decomposition, reranking và chọn kết quả đa dạng.
- Xác định khi nào graph hoặc multimodal retrieval là cần thiết.
- Thiết kế ablation để quy kết thay đổi về quality, latency và cost.

## Kiến thức tiên quyết

Hoàn thành [kỹ thuật retrieval](05-retrieval-engineering.md), [lab RAG](10-rag-system-worked-lab.md) và [đánh giá RAG](09-rag-evaluation-and-observability.md).

## 1. Trước hết chẩn đoán retrieval failure

Gán nhãn lỗi theo nguyên nhân:

- tài liệu liên quan không có trong corpus;
- parser bỏ sót văn bản, ô bảng, caption hoặc layout;
- chunk tách định nghĩa khỏi điều kiện/heading;
- query dùng thuật ngữ khác với tài liệu;
- passage liên quan được retrieve nhưng xếp quá thấp;
- passage liên quan được retrieve nhưng mất khi context packing;
- evidence mâu thuẫn, lỗi thời hoặc user không được quyền đọc.

Mỗi nguyên nhân gợi ý một can thiệp khác nhau. Tăng số chunk truy xuất không sửa được tài liệu chưa từng được index.

## 2. Chiến lược chunking và indexing

### Chunk cố định hoặc đệ quy

Dùng làm baseline đơn giản cho văn xuôi tương đối đồng nhất. Giữ ranh giới đoạn và heading khi có thể. Đo kích thước theo cách biểu diễn model sử dụng, không chỉ theo ký tự.

### Semantic chunking

Dùng thay đổi chủ đề hoặc cấu trúc diễn ngôn để chọn ranh giới. Cách này giữ được ý tưởng mạch lạc nhưng thêm model/heuristic và có thể tạo chunk kích thước không đồng đều. So với splitter đơn giản trên câu hỏi có nhãn.

### Parent-child retrieval

Index child chunk nhỏ để khớp chính xác, sau đó trả parent section lớn hơn để cung cấp ngữ cảnh. Lưu ID và provenance của cả hai để có thể mở rộng child mà không mất vị trí nguồn.

### Summary hoặc hierarchical index

Index summary ở cấp document/section để định tuyến câu hỏi rộng, rồi truy xuất passage chi tiết làm evidence. Summary là công cụ tìm đường, không thay thế evidence gốc.

### Late chunking

Với embedding system tương thích, encode một vùng context lớn trước khi pooling biểu diễn token thành vector chunk nhỏ. Điều này giữ được ngữ cảnh lân cận nhưng phụ thuộc model hỗ trợ, memory và ánh xạ vector trở lại source span chính xác.

## 3. Đánh đổi Approximate Nearest Neighbor

Vector collection lớn thường dùng approximate nearest-neighbor (ANN) index. Hierarchical Navigable Small World (HNSW) xây graph để tìm các vùng lân cận tiềm năng thay vì so sánh mọi vector.

Các tham số HNSW biểu thị đánh đổi:

- **M:** độ kết nối graph; giá trị cao thường dùng thêm memory và có thể cải thiện khả năng điều hướng.
- **ef_construction:** độ rộng candidate trong lúc xây graph; giá trị cao tốn thời gian build hơn và có thể tăng chất lượng index.
- **ef_search:** độ rộng candidate lúc query; giá trị cao thường cải thiện recall, đổi lại query tốn thêm chi phí.

Tên và default parameter thay đổi tùy vector store. Đo recall so với exact search trên mẫu đại diện trước khi tuning index. Báo cáo memory, build time, query latency và recall cùng nhau.

## 4. Hybrid lexical và dense retrieval

Dense embedding khớp paraphrase nhưng có thể bỏ identifier chính xác, tên hiếm, error code và chuỗi lạ. Lexical retrieval như BM25 mạnh với từ chính xác nhưng có thể bỏ lỡ paraphrase ngữ nghĩa.

Baseline vững thường chạy cả hai rồi gộp danh sách xếp hạng. Reciprocal Rank Fusion (RRF) tính:

$$
\operatorname{RRF}(d)=
\sum_{r\in\mathcal R}\frac{1}{k+\operatorname{rank}_r(d)},
$$

trong đó $\mathcal R$ là tập danh sách kết quả đã xếp hạng và $k$ làm giảm ảnh hưởng của vị trí hạng rất cao. RRF không cần hiệu chỉnh raw score giữa các retriever.

Đánh giá lexical, dense và hybrid trên cùng query set. Giữ score và rank từ từng channel để debug.

## 5. Query transformation

### Sinh nhiều query

Tạo một vài cách diễn đạt tìm kiếm khác rồi gộp kết quả. Cách này có thể tăng recall khi lệch vocabulary, nhưng cũng tạo query thừa và tài liệu trùng.

### Hypothetical Document Embeddings (HyDE)

Yêu cầu model viết passage giả định có thể trả lời query, sau đó dùng embedding của nó làm query retrieval. Passage giả định chỉ là công cụ tìm kiếm, **không phải evidence**. Chi tiết bịa trong đó tuyệt đối không được hiển thị như dữ kiện truy xuất.

### Query decomposition

Tách câu hỏi nhiều phần thành subquestion có thể trả lời, retrieve evidence cho mỗi phần và kiểm tra đủ mọi ý cần thiết trước khi tổng hợp. Giữ query gốc để decomposition không âm thầm thay đổi ý định user.

### Step-back và routing

Query step-back rộng hơn có thể tìm khái niệm nền; router có thể chọn collection hoặc retriever theo loại tác vụ. Chỉ thêm khi evaluation riêng cho thấy có ích và tồn tại fallback an toàn.

## 6. Xử lý sau retrieval

### Cross-encoder reranking

Bi-encoder embed query và document riêng, hiệu quả với tập candidate lớn. Cross-encoder chấm cặp query-document cùng nhau, có thể xếp hạng tập candidate nhỏ chính xác hơn nhưng tốn chi phí hơn cho mỗi cặp.

Pipeline hai giai đoạn:

1. retrieve tập candidate rộng bằng phương pháp rẻ;
2. rerank số candidate giới hạn;
3. trả evidence hữu ích nhất trong context budget.

Đo mức cải thiện so với latency và cost thêm vào.

### Maximal Marginal Relevance (MMR)

MMR cân bằng relevance với query và độ đa dạng giữa passage được chọn:

$$
\operatorname{MMR}(d)=
\lambda\,\operatorname{sim}(q,d)
-(1-\lambda)\max_{s\in S}\operatorname{sim}(d,s),
$$

trong đó $S$ là tập đã chọn. $\lambda$ lớn ưu tiên relevance trực tiếp; $\lambda$ nhỏ phạt trùng lặp mạnh hơn. Context đa dạng chỉ có ích nếu vẫn giữ đủ evidence cần thiết.

### Contextual compression và ordering

Compression có thể bỏ câu không liên quan trong passage, nhưng cũng có thể làm mất phủ định, ngày tháng, ngoại lệ hoặc citation anchor. Giữ passage gốc và so sánh với evidence đã nén.

Thử thứ tự context. Một số model dùng thông tin đầu/cuối context dài đáng tin hơn thông tin bị chôn ở giữa. Điều này phụ thuộc model và prompt; không coi một quy tắc ordering là phổ quát.

## 7. Graph retrieval

Graph-based retrieval có ích khi câu hỏi phụ thuộc entity và relation giữa nhiều tài liệu: sở hữu, chuỗi phụ thuộc, dòng thời gian hoặc kết nối nhiều bước.

Graph RAG thường:

1. trích entity và relation từ tài liệu nguồn;
2. xác minh định danh entity và provenance;
3. lưu graph có liên kết về source span;
4. ánh xạ câu hỏi thành graph query hoặc traversal giới hạn;
5. truy xuất evidence nguồn cho entity/relation kết quả;
6. sinh câu trả lời có citation.

Entity extraction có thể sai, entity resolution có thể gộp nhầm người khác nhau và khó đảm bảo graph đầy đủ. Mọi edge dùng trong câu trả lời phải truy về source evidence. Nhiều tác vụ tra cứu sự kiện trực tiếp không cần graph retrieval.

## 8. Multimodal retrieval

Index chỉ-văn-bản làm mất thông tin ở diagram, form scan, chart và image. Các cách có thể dùng:

- OCR và trích layout;
- image caption được index như văn bản;
- shared image-text embedding;
- visual-language retrieval trên ảnh trang hoặc crop.

Chọn cách đơn giản nhất vẫn giữ được evidence cần cho tác vụ. OCR có thể mất quan hệ không gian; caption có thể bỏ nhãn nhỏ; image embedding khó diễn giải/citation hơn. Giữ provenance page, crop và bounding box để reviewer kiểm tra bằng mắt.

## 9. Thứ tự tối ưu có kiểm soát

Mỗi lần đổi một layer:

1. sửa parsing và metadata;
2. so sánh ranh giới chunk;
3. so lexical, dense và hybrid retrieval;
4. tune candidate count và ANN recall;
5. thử query transformation;
6. rerank và loại trùng;
7. chỉnh context packing và ordering;
8. chỉ thử graph hoặc multimodal với nhóm query phù hợp.

Với mỗi thay đổi, báo cáo retrieval recall/precision, answer grounding/relevance, latency, token use, cost và error category. Giữ slice câu hỏi không trả lời được và authorization trong evaluation set.

## Bài tập có đáp án

1. Vector search bỏ sót product ID chính xác. Nên so sánh channel nào?
2. Passage HyDE có ngày tháng nghe hợp lý nhưng không có nguồn. Câu trả lời có được citation ngày đó không?
3. Khi nào parent-child retrieval hữu ích?
4. Đánh đổi chính của cross-encoder reranking là gì?
5. Loại câu hỏi nào phù hợp với graph retrieval?
6. Vì sao cần giữ source span sau contextual compression?

### Đáp án

1. So lexical/BM25 retrieval và hybrid merge với dense retrieval.
2. Không. Đoạn được sinh ra chỉ là công cụ tạo query, không phải evidence nguồn.
3. Khi chunk nhỏ giúp matching tốt nhưng cần section lớn hơn để có context đủ trả lời.
4. Xếp hạng query-document chung chính xác hơn trên tập candidate nhỏ, đổi lại compute và latency cao hơn cho mỗi cặp.
5. Câu hỏi cần quan hệ giữa nhiều entity/tài liệu, ví dụ lần theo dependency hoặc ownership theo thời gian.
6. Reviewer cần xác minh câu trả lời theo bản gốc; compression có thể bỏ qualifier hoặc citation anchor.

## Mini-project

Trên một corpus, so sánh dense baseline dùng recursive chunking, hybrid retrieval với RRF và một phương pháp post-retrieval. Chỉ thêm semantic hoặc parent-child chunking nếu nhóm lỗi có nhãn gợi ý nhu cầu. Báo cáo query set cố định, retrieval metric riêng với answer metric, p95 latency, cost và ít nhất mười lỗi retrieved-context.

## Lỗi thường gặp

- Thêm retrieval nâng cao trước khi đo baseline failure.
- Coi hypothetical answer sinh ra là evidence.
- So raw score giữa retriever khác nhau mà không hiệu chỉnh.
- Báo answer quality nhưng không có retrieval recall.
- Dùng GraphRAG cho tra cứu đơn giản mà chưa chứng minh cần thiết.
- Bỏ qua index-build cost, memory, access filtering và citation provenance.
