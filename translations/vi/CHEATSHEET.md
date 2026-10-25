# Cheat Sheet Học AI

Tài liệu tra nhanh khái niệm, công thức, metric và quy trình trong repository. Dùng để nhớ lại sau khi đã học bài chi tiết; cheat sheet không thay thế ví dụ giải từng bước và bài tập.

## 1. Workflow ML đáng tin cậy

1. **Định nghĩa tác vụ:** biết gì tại thời điểm dự đoán, cần output nào, ai dùng và lỗi gây thiệt hại gì.
2. **Kiểm tra data:** ý nghĩa row, schema, unit, missingness, label, duplicate, bias thu thập và license.
3. **Chia tập trước khi fit transform:** chọn random, group hoặc time-aware split sát deployment.
4. **Tạo baseline đơn giản:** majority class, mean/median, domain rule, lexical search hoặc model nhỏ.
5. **Dựng pipeline:** chỉ fit imputation, scaling, feature selection và model trên training fold.
6. **Evaluation bằng metric phù hợp:** dùng validation để lựa chọn; giữ test cho ước lượng cuối.
7. **Xem lỗi và subgroup:** tìm pattern thay vì chỉ xem một score tổng hợp.
8. **Ghi và tái lập:** data version, code, setting, dependency, model và preprocessing.
9. **Deploy có kiểm tra:** validate input, đo latency/cost, monitor quality và giữ rollback.

## 2. Toán cần nhớ

### Đại số tuyến tính

Dot product:

$$
\mathbf w^\top\mathbf x=\sum_{j=1}^{d}w_jx_j
$$

Euclidean norm:

$$
\|\mathbf x\|_2=\sqrt{\sum_{j=1}^{d}x_j^2}
$$

Cosine similarity so hướng của hai vector:

$$
\cos(\mathbf u,\mathbf v)=
\frac{\mathbf u^\top\mathbf v}{\|\mathbf u\|_2\|\mathbf v\|_2}
$$

Shape phép nhân ma trận: $(n,d)@(d,k)\rightarrow(n,k)$. Dự đoán tuyến tính: $\hat y=\mathbf w^\top\mathbf x+b$; cho batch: $\hat{\mathbf y}=X\mathbf w+b$. Thường row là example, column là feature nhưng cần kiểm tra quy ước từng library.

### Calculus và optimization

Derivative là tốc độ biến thiên cục bộ. Gradient là vector partial derivative, chỉ hướng local increase lớn nhất.

Gradient descent:

$$
\theta\leftarrow\theta-\eta\nabla_\theta J(\theta)
$$

Regularized objective:

$$
J_{\mathrm{total}}=J_{\mathrm{data}}+\lambda\Omega(\theta)
$$

Chain rule với $f(g(x))$: $\frac{d}{dx}f(g(x))=f'(g(x))g'(x)$.

Nếu loss diverge, trước tiên xem learning rate, input scale, giá trị không hợp lệ và gradient magnitude. Train cải thiện mà validation xấu đi thì kiểm tra overfitting hoặc split lệch deployment.

### Probability và statistics

$$
P(\neg A)=1-P(A),\qquad
P(A\mid B)=\frac{P(A\cap B)}{P(B)}
$$

Bayes:

$$
P(A\mid B)=\frac{P(B\mid A)P(A)}{P(B)}
$$

Mean và variance:

$$
\bar x=\frac1n\sum_{i=1}^{n}x_i,\qquad
\operatorname{Var}(X)=\mathbb{E}[(X-\mathbb{E}[X])^2]
$$

Standard deviation là căn bậc hai variance. Sample lớn vẫn có thể bias; correlation tự nó không chứng minh causation.

## 3. Nhắc nhở xử lý dữ liệu

- Missing value có ý nghĩa; tìm lý do thiếu trước khi thay thế.
- Chỉ fit imputer, scaler, encoder, feature selection và PCA trên train.
- Random split: example độc lập. Group split: example liên quan ở cùng một phía. Time split: train quá khứ, đánh giá tương lai.
- Câu hỏi chống leakage: “Feature này có sẵn ở thời điểm dự đoán thật không?”
- Giữ raw data nguyên vẹn; ghi quyết định cleaning, nguồn và license.
- Kiểm tra pandas ban đầu: shape, dtypes, head, tổng null và unique count.
- Trước SQL join, kiểm tra key uniqueness và row count. WHERE lọc row; GROUP BY tổng hợp; HAVING lọc group; dùng IS NULL cho giá trị thiếu.
- CSV dễ trao đổi; Parquet lưu kiểu/column hiệu quả; JSON hợp record lồng nhau. Chọn theo workload, giữ provenance nguồn thô.
- Với streaming/time series, phân biệt event time với arrival time; feature chỉ dùng thông tin sẵn có trước prediction cutoff.
- Với join, nêu grain mỗi bảng và cardinality mong đợi; sau đó kiểm tra row count và aggregate total.
- Pipeline định kỳ nên deterministic/idempotent, có watermark, quality gate, lineage và output có version.
- Offline training và online serving phải dùng feature definition giống nghĩa và point-in-time-correct.

## 4. Software engineering essentials

- Tách ingestion, feature logic, training, inference, API và configuration thành module rõ.
- Dùng environment, dependency record, Git diff, structured log, typed interface và error handling tập trung.
- Tầng test: unit → schema/contract → integration → end-to-end. Model evaluation là kiểm tra thống kê bổ sung cho software test.
- Với mỗi model API, định nghĩa field/type/unit, xử lý missing, nghĩa label, timeout và error response.
- Retry cần giới hạn/backoff; side-effect action cần idempotency key và authorization.

## 5. Python và cấu trúc dữ liệu

| Cấu trúc/thuật toán | Thời gian thường gặp | Ghi nhớ |
| --- | --- | --- |
| List index | $O(1)$ | Truy cập theo vị trí |
| List search | $O(n)$ | Duyệt tuyến tính |
| List append | amortized $O(1)$ | Thêm cuối danh sách |
| Insert/remove đầu list | $O(n)$ | Các phần tử sau cần dịch |
| Dict/set lookup/insert/delete | trung bình $O(1)$, worst $O(n)$ | Hash table |
| Deque thêm/bỏ hai đầu | $O(1)$ | Dùng làm queue |
| Heap đọc min | $O(1)$ | Phần tử nhỏ nhất ở gốc |
| Heap push/pop | $O(\log n)$ | Priority queue |
| Linked list search | $O(n)$ | Không có random indexing |
| Linked list insert đầu | $O(1)$ | Chỉ khi đã có node/đầu |
| Balanced BST search/insert/delete | $O(\log n)$ | BST lệch có thể thành $O(n)$ |
| Graph adjacency list | space $O(V+E)$ | BFS/DFS $O(V+E)$ |
| Linear search | $O(n)$ | Không cần sắp xếp |
| Binary search | $O(\log n)$ | Input đã sort và truy cập ngẫu nhiên |
| Comparison sort tổng quát | $O(n\log n)$ | Sort quadratic chủ yếu để học/input nhỏ |

Time/space complexity mô tả tốc độ tăng theo kích thước input, không phải số giây chính xác trên máy cụ thể. Xem [lộ trình Python, OOP và DSA](curriculum/02-python/README.md).

## 6. Machine Learning

### Tác vụ thường gặp

- **Regression:** dự đoán số.
- **Classification:** dự đoán category hoặc xác suất category.
- **Clustering:** nhóm example không nhãn; cần diễn giải/validate cluster.
- **Dimensionality reduction:** tạo representation ít chiều; có thể mất thông tin/khả năng diễn giải.

### Loss và metric

Mean squared error và mean absolute error:

$$
\operatorname{MSE}=\frac1n\sum_{i=1}^{n}(y_i-\hat y_i)^2,\qquad
\operatorname{MAE}=\frac1n\sum_{i=1}^{n}|y_i-\hat y_i|
$$

Precision/recall/F1:

$$
P=\frac{TP}{TP+FP},\qquad
R=\frac{TP}{TP+FN},\qquad
F_1=\frac{2PR}{P+R}
$$

Accuracy:

$$
\operatorname{Accuracy}=\frac{TP+TN}{TP+TN+FP+FN}
$$

Accuracy có thể gây hiểu lầm khi class imbalance. IoU là intersection area chia union area, dùng cho bounding box/mask.

**TP** = true positive; **FP** = false positive; **FN** = false negative; **TN** = true negative.

Chọn metric và classification threshold theo chi phí lỗi. Báo count và lỗi từng class bên cạnh metric tổng hợp. Tránh tune lặp trên test cuối.

### Trực giác model

- Linear/logistic model: baseline nhanh, đơn giản; diễn giải coefficient cẩn thận.
- Decision tree: nonlinear split dạng quy tắc; tree sâu dễ overfit.
- Random forest: lấy trung bình nhiều tree được random hóa.
- Gradient boosting: thêm learner tuần tự để sửa lỗi trước.
- k-means: gán điểm cho một trong $k$ center; scale feature và kiểm tra giả định.
- PCA: tìm hướng variance cao; hữu ích nén dữ liệu nhưng không tự động tăng predictive power.

## 7. Deep Learning

Neuron:

$$
z=\mathbf w^\top\mathbf x+b,\qquad a=\phi(z)
$$

Softmax và cross-entropy:

$$
p_i=\frac{\exp(z_i)}{\sum_j\exp(z_j)},\qquad
\mathcal L=-\sum_i y_i\log p_i
$$

Training step: forward pass → loss → backpropagate gradient → optimizer update. Batch là số example mỗi update; epoch là một lượt qua training data.

- CNN dùng chung local filter để bắt pattern không gian.
- Transformer attention: $\operatorname{softmax}(QK^\top/\sqrt{d_k})V$; trộn thông tin theo similarity học được.
- Luôn so với baseline và theo dõi validation, không chỉ training loss.

## 8. Computer Vision

- Image array thường có shape $(height,width,channels)$; indexing pixel thường là $[row,column]=[y,x]$.
- Kiểm tra channel order (RGB/BGR), value range, resize, crop và normalization.
- **Classification:** một/nhiều nhãn cho image.
- **Detection:** class và bounding box; IoU đo overlap; AP tóm tắt precision-recall curve.
- **Semantic segmentation:** class cho mỗi pixel; **instance segmentation** tách từng object cá thể.
- Phải biến đổi label đồng bộ với image. Để frame gần nhau hoặc cùng scene trong một split nếu có leakage nguy cơ.

## 9. 3D Vision và Robotics

- Ghi tên từng coordinate frame và unit. $T_{A\leftarrow B}$ đổi điểm từ frame B sang A.
- Homogeneous transform: $T=\begin{bmatrix}R&t\\0&1\end{bmatrix}$; thứ tự composition quan trọng.
- Pinhole projection:

$$
u=f_x\frac{X}{Z}+c_x,\qquad
v=f_y\frac{Y}{Z}+c_y
$$

- Back-project depth:

$$
X=\frac{(u-c_x)Z}{f_x},\qquad
Y=\frac{(v-c_y)Z}{f_y}
$$

- RGB-D/point cloud có thể chứa depth sai, motion và calibration error.
- Giữ timestamp/frame ID. Perception output không phải motion command an toàn; kiểm tra confidence, workspace, constraint và latency.

## 10. NLP và LLM System

- Token là đơn vị đầu vào model; token ID không phải khoảng cách ngữ nghĩa.
- Transformer dùng attention và feed-forward layer; causal LM dự đoán token kế tiếp.
- Temperature/top-p ảnh hưởng cách sampling, không quyết định độ thật.
- RAG: chuẩn bị/index tài liệu → retrieve → tùy chọn rerank → trả lời theo evidence → validate citation.
- Đánh giá retrieval riêng với answer quality; đưa câu hỏi không có đáp án và đo abstention phù hợp.
- User input và retrieved text không tin cậy. Enforce access control trong code, không chỉ prompt.
- Prompting đổi instruction; RAG cung cấp evidence hiện hành; fine-tuning điều chỉnh behavior/format. Chọn theo nguồn lỗi đã đo.
- Đánh giá extraction, retrieval, generation, tool, privacy, latency và cost trong cùng user task.

## 11. Reinforcement Learning

MDP gồm state $\mathcal S$, action $\mathcal A$, transition $P$, reward $R$, discount $\gamma$.

Discounted return:

$$
G_t=r_{t+1}+\gamma r_{t+2}+\gamma^2r_{t+3}+\cdots
$$

- $V(s)$: expected future return từ state $s$.
- $Q(s,a)$: expected return khi làm action $a$ ở state $s$.
- Epsilon-greedy khám phá ngẫu nhiên xác suất $\epsilon$, còn lại chọn action tốt nhất đã biết.
- Đánh giá nhiều seed, so baseline, kiểm tra trajectory và thử reward loophole.

Q-learning update:

$$
Q(s,a)\leftarrow Q(s,a)+\alpha\left[r+\gamma\max_{a'}Q(s',a')-Q(s,a)\right]
$$

## 12. Agentic AI

- Ưu tiên workflow cố định nếu biết trước các bước; dùng agent khi model-directed choice tạo giá trị đo được.
- Định nghĩa task boundary, tool được phép, input/output có kiểu, max step, timeout, cost budget và stop rule.
- Validate tool argument/result trong program code. Dùng least privilege, idempotency và error rõ.
- Coi tool output/tài liệu retrieved là input không tin cậy; prompt injection là vấn đề security hệ thống.
- Đo task success, tool correctness, action sai quyền, recovery, latency, cost và safety failure.
- Không tuyên bố hoàn tất nếu thiếu evidence tool đã thành công.
- **Skill** = quy trình/instruction tái sử dụng; **tool** = capability thực thi; **memory/state** = thông tin giữ lại; **workflow** = control flow; **harness** = runtime/context/policy quanh agent.
- Agent Skills thường đóng gói SKILL.md cùng reference/asset/script tùy chọn. Review script và permission; skill không phải sandbox.
- LangChain create_agent là model/tool loop cấp cao; LangGraph cung cấp graph state, node, route, checkpoint và pause/resume tường minh.
- MCP chuẩn hóa việc khám phá/gọi tool và truy cập resource/prompt; nó không cấp quyền hay đảm bảo an toàn.
- Multi-agent pattern gồm router/handoff, manager-as-tool, sequential pipeline, parallel fan-out/fan-in. So từng cách với baseline một agent.

## 13. MLOps: checklist nhanh

- Version hóa data, code, dependency, preprocessing, model và configuration.
- Validate inference input schema; đồng bộ transform train/serve.
- Đo latency percentile, throughput, memory, error và data quality.
- Drift là tín hiệu điều tra, không phải bằng chứng model hỏng; input ổn định không chứng minh performance tốt.
- Định nghĩa alert owner, response, rollback, privacy, retention và bảo trì.

## 14. Responsible AI

- Xác định ai hưởng lợi, ai có thể chịu hại và nơi hệ thống không phù hợp.
- Đo subgroup liên quan kèm sample size/uncertainty; metric tổng thể có thể che lỗi lệch.
- Fairness criteria thể hiện mục tiêu khác nhau và có thể mâu thuẫn; nêu trade-off đã chọn.
- Giảm thiểu data, kiểm tra permission/retention, bảo vệ log và artifact.
- Human oversight cần có thẩm quyền và đường appeal/correction dùng được.
- Đánh giá lại rủi ro khi data, model, interface hoặc deployment context thay đổi.

## 15. Câu hỏi debug khi kết quả sai

1. Label có khớp example không?
2. Shape, type, unit và coordinate frame có đúng không?
3. Có leakage hoặc split khác deployment không?
4. Baseline đơn giản có hoạt động không?
5. Preprocessing và inference có giống nhau không?
6. Example nào thất bại; label/data có đáng tin không?
7. Người khác có tái lập được run và metric không?

## Bài học liên quan

Mở [bản đồ curriculum](curriculum/README.md) để xem giải thích và bài tập chi tiết theo từng chủ đề.
