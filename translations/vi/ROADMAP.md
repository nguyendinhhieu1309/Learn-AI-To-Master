# Lộ trình Học tập

Lộ trình tự học. Chuyển tiếp khi bạn có thể giải thích ý tưởng bằng lời của mình và làm checkpoint mà không chép lời giải. Quay lại chủ đề cũ bất cứ khi nào bài sau để lộ lỗ hổng kiến thức.

## Lộ trình nền tảng

### 0. Thiết lập cách học

Đọc [cách sử dụng lộ trình](curriculum/00-start-here/01-how-to-use-this-path.md). Cài Python, editor/notebook, Git và nơi lưu ghi chú.

**Checkpoint:** chạy script Python ngắn, lưu ghi chú Markdown và tạo commit Git nhỏ.

### 1. Học phần toán cần dùng

Học [Toán cho AI](curriculum/01-math/README.md) song song với lập trình. Trước hết tập trung vector, matrix, function, derivative, probability và summary statistic.

**Checkpoint:** giải thích dot product, derivative, mean và probability bằng ví dụ số nhỏ.

### 2. Học Python và software engineering

Theo [Python và computing](curriculum/02-python/README.md). Học project structure, environment, Git, testing, API và data contract trong [AI Engineering](curriculum/11-ai-engineering/README.md).

**Checkpoint:** đóng gói chương trình Python đọc CSV, tính summary, validate input, log context hữu ích và có hướng dẫn chạy rõ.

### 3. Học làm việc với dữ liệu

Dùng [Xử lý dữ liệu và data engineering](curriculum/03-data/README.md) để đi từ raw table tới dataset cho model và pipeline ingestion/feature tái lập. Thực hành tabular data trước text, image, time series hay 3D.

**Checkpoint:** tạo workflow data-cleaning tái lập, ghi giả định và kiểm tra leakage.

### 4. Học classical machine learning

Theo [Machine Learning](curriculum/04-machine-learning/README.md). So baseline đơn giản với linear model, tree và unsupervised method. Học cách metric và data split ảnh hưởng kết luận.

**Checkpoint:** train ít nhất hai model, đánh giá trên data giữ lại và giải thích từng model thất bại ở đâu.

### 5. Học deep learning

Tiếp tục với [Deep Learning](curriculum/05-deep-learning/README.md). Hiểu tensor, loss function, gradient descent và training diagnostic trước khi chuyên sâu.

**Checkpoint:** train neural network nhỏ, xem learning curve, lỗi và setting tái lập.

## Chọn một chuyên ngành

Khi đã vững lộ trình nền tảng, chọn trước một nhánh; có thể quay lại nhánh khác sau.

- **Vision và robotics:** [Computer Vision](curriculum/06-computer-vision/README.md) → [3D Vision và robotics](curriculum/07-3d-vision-robotics/README.md) → [AI Engineering](curriculum/11-ai-engineering/README.md).
- **Ngôn ngữ và agent:** [NLP, LLM và Generative AI](curriculum/08-nlp-llms/README.md) → [Agentic AI](curriculum/10-agentic-ai/README.md) → evaluation và deployment.
- **ML/data engineering:** [Xử lý dữ liệu và data engineering](curriculum/03-data/README.md) → [Machine Learning](curriculum/04-machine-learning/README.md) → [AI Engineering](curriculum/11-ai-engineering/README.md).
- **Ra quyết định:** [Reinforcement Learning](curriculum/09-reinforcement-learning/README.md) → project simulation → evaluation và triển khai an toàn.

Ở mọi nhánh, học [Responsible AI](curriculum/12-responsible-ai/README.md) để xem ai được lợi, ai có thể bị hại, cách dùng data nào phù hợp và con người review/khiếu nại outcome ra sao.

## Trình tự project gợi ý

1. **Nền tảng software:** đóng gói Python data tool nhỏ với CLI hoặc API.
2. **Data report:** xây ingestion/cleaning pipeline tái lập; giải thích vấn đề chất lượng và giới hạn.
3. **Prediction baseline:** so rule đơn giản, linear model và tree model bằng validation strategy đúng.
4. **Ứng dụng deep learning:** train/adapt image/text model, phân tích lỗi và so baseline đơn giản.
5. **LLM application:** xây retrieval system có grounding, đánh giá riêng retrieval/answer và có case không thể trả lời.
6. **Agent system:** xây bounded agent có reusable skill, tool hẹp, harness, trace và human approval cho action hệ trọng.
7. **Phân phối tin cậy:** deploy project có observability, cost control, privacy/security note và rollback.

Dùng [Core AI Engineer Capstone](projects/CORE_AI_ENGINEER_CAPSTONE.md) làm hướng dẫn triển khai/review chi tiết cho bốn giai đoạn đầu và lần bàn giao cuối.

## Nhịp học

Với mỗi bài: đọc, tự chạy ví dụ, giải bài tập không nhìn lời giải rồi viết tóm tắt ngắn. Dùng [cheat sheet](CHEATSHEET.md) để nhớ lại sau khi đã hiểu derivation. Với project: định nghĩa problem/metric trước, giữ baseline, ghi thay đổi và trình bày cả experiment thành công lẫn thất bại.

## Checklist sẵn sàng cho AI Engineer

Bạn có thể bắt đầu project AI đầu-cuối nhỏ khi làm được:

- triển khai và giải thích đường đi từ nguồn data tới prediction;
- tạo split hợp lệ và bảo vệ lựa chọn metric/baseline;
- đóng gói code để developer khác tái lập;
- đưa hành vi model ra input/output contract rõ;
- chẩn đoán riêng lỗi model, data, software và infrastructure;
- theo dõi latency, error, cost, data change và task quality;
- nêu giới hạn privacy, security, fairness và deployment.
