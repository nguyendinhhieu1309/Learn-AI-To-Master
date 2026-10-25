# NLP, LLM và AI tạo sinh

## Kết quả học tập

- Biểu diễn văn bản, hiểu tokenization, embeddings và các baseline NLP cổ điển.
- Giải thích transformer, quá trình huấn luyện language model, inference và model adaptation.
- Xây dựng, đánh giá ứng dụng retrieval-augmented và generative AI.
- Thực hành an toàn, kiểm soát chi phí và đánh giá trong toàn bộ vòng đời LLM.

## Kiến thức tiên quyết

Python, xử lý dữ liệu và kiến thức ML cơ bản. Deep learning và probability giúp hiểu sâu transformer training và fine-tuning.

## Lộ trình gợi ý

1. [Biểu diễn văn bản và Transformer](01-text-representations-and-transformers.md)
2. [NLP cổ điển, sequence model và evaluation](02-classical-nlp-sequence-models-and-evaluation.md)
3. [Nền tảng LLM và inference](03-llm-foundations-and-inference.md)
4. [Thiết kế prompt và structured output](04-prompt-design-and-structured-outputs.md)
5. [Kỹ thuật retrieval](05-retrieval-engineering.md)
6. [Fine-tuning và model adaptation](06-fine-tuning-and-model-adaptation.md)
7. [Evaluation, an toàn và chi phí LLM](07-llm-evaluation-safety-and-cost.md)
8. [RAG, evaluation và an toàn](08-rag-evaluation-and-safety.md)
9. [Đánh giá RAG và observability](09-rag-evaluation-and-observability.md)
10. [Lab xây dựng hệ thống RAG](10-rag-system-worked-lab.md)
11. [Các retrieval pattern RAG nâng cao](11-advanced-rag-retrieval-patterns.md)

## Bài thực hành

- Xây baseline TF-IDF cho một bài toán phân loại văn bản.
- So sánh tokenization trên mã nguồn, văn bản đa ngôn ngữ và từ hiếm.
- Lập chỉ mục tìm kiếm nhỏ; kiểm tra passages được truy xuất trước khi sinh câu trả lời.
- Viết test case cho câu hỏi không có đáp án, câu hỏi mơ hồ và prompt injection.
- So sánh classifier TF-IDF với transformer trên cùng một data split.
- Mỗi lần chỉ thay đổi một retrieval setting; đo cả chất lượng câu trả lời lẫn retrieval recall.
- Hoàn thành [lab RAG](10-rag-system-worked-lab.md), gồm permission, câu hỏi không có đáp án và kiểm tra citation.
- Hoàn thành [bài so sánh RAG nâng cao](11-advanced-rag-retrieval-patterns.md) trên query set cố định trước khi chọn thiết kế phức tạp hơn.

## Dự án checkpoint

Tạo công cụ hỏi đáp trên một tập tài liệu nhỏ có nguồn gốc rõ ràng. Đo retrieval riêng với chất lượng câu trả lời, trích dẫn passages nguồn và ghi rõ tình huống hệ thống cần từ chối trả lời.

So sánh prompting, retrieval và fine-tuning theo yêu cầu nhiệm vụ; ghi nhận chất lượng, latency và chi phí của từng cách.
