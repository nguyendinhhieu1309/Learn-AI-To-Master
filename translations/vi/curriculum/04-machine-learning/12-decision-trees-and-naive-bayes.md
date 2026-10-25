# Decision tree và Naive Bayes

[Xem bài gốc bằng tiếng Anh](../../../../curriculum/04-machine-learning/12-decision-trees-and-naive-bayes.md)

Decision tree tạo quy tắc bằng cách chia feature space. Naive Bayes so sánh xác suất class bằng Bayes' rule. Chúng có giả định khác nhau và thường là baseline nhanh, hữu ích trước khi cân nhắc model phức tạp hơn.

## Mục tiêu học tập

- Tính entropy, Gini impurity và information gain.
- Giải thích recursive splitting, stopping và pruning.
- Phân biệt một tree, random forest và gradient-boosted tree.
- Suy ra class score của Naive Bayes và hiểu smoothing.
- Nhận biết giả định cùng failure case của hai họ phương pháp.

## 1. Tree đặt chuỗi câu hỏi

Decision tree đặt câu hỏi như “tuổi tài khoản dưới 30 ngày không?” Mỗi internal node áp dụng phép thử feature; mỗi leaf xuất phân phối class hoặc dự đoán số. Đường đi từ root đến leaf là quy tắc dễ diễn giải, nhưng tree lớn khó hiểu.

## 2. Đo impurity của class

Giả sử node có tỷ lệ class $p_1,\ldots,p_K$. Entropy:

$$
H=-\sum_{k=1}^{K}p_k\log_2p_k,
$$

quy ước $0\log 0=0$. Gini impurity:

$$
G=1-\sum_{k=1}^{K}p_k^2.
$$

Cả hai bằng 0 khi mọi ví dụ cùng nhãn và đạt lớn nhất khi class trộn đều. Với split thành child node $c$, dùng impurity có trọng số theo kích thước:

$$
I_{\mathrm{after}}=\sum_c\frac{n_c}{n}I(c).
$$

Information gain bằng entropy của parent trừ entropy trung bình có trọng số của child. Tree chọn feature/threshold có gain lớn nhất (hoặc giảm impurity nhiều nhất).

### Ví dụ split có hướng dẫn

Node có bốn ví dụ positive và bốn negative, entropy là $H(0.5,0.5)=1$ bit. Một split tạo child thuần size 4 và child khác có hai positive, hai negative. Entropy child có trọng số là $\tfrac48(0)+\tfrac48(1)=0.5$, nên information gain là $1-0.5=0.5$ bit. Split làm cả hai child thuần sẽ gain 1 bit.

## 3. Mở rộng, dừng, pruning

Greedy tree lặp lại việc chọn split tốt nhất cục bộ. Tree quá sâu có thể overfit, nhất là khi leaf nhỏ. Kiểm soát complexity bằng maximum depth, minimum leaf size, minimum impurity decrease hoặc cost-complexity pruning. Tune bằng validation split/cross-validation; chốt tree cuối trước khi đánh giá test.

Tree xử lý interaction phi tuyến và scale hỗn hợp mà không cần standardize feature. Một tree đơn thường bất ổn: thay đổi nhỏ trong data có thể đổi split sớm. Tree cũng có thể ưu ái một số cơ hội split và tạo importance gây hiểu nhầm khi feature tương quan hoặc có nhiều điểm chia tiềm năng.

## 4. Ensemble tree

- **Bagging / random forest:** training nhiều tree trên bootstrap sample và feature subset ngẫu nhiên, rồi lấy trung bình/bỏ phiếu. Chủ yếu giảm variance và vững trên dữ liệu dạng bảng.
- **Gradient boosting:** lần lượt thêm tree để giảm loss, thường dựa trên residual hoặc gradient. Có thể đạt accuracy cao nhưng cần regularization và tuning cẩn thận.
- **Extremely randomized tree:** tăng tính ngẫu nhiên của split, đánh đổi một phần bias để giảm variance.

Ensemble khó diễn giải trực tiếp hơn một tree nhỏ. Ngoài score tổng, so sánh calibration, latency, memory và error slice. Feature importance mô tả liên hệ với hành vi model, không phải giải thích nhân quả.

## 5. Naive Bayes từ Bayes' rule

Với class $c$ và feature vector $x=(x_1,\ldots,x_d)$:

$$
P(c\mid x)\propto P(c)P(x\mid c).
$$

Naive Bayes giả định feature độc lập có điều kiện khi biết class:

$$
P(x\mid c)=\prod_{j=1}^{d}P(x_j\mid c).
$$

Do đó so sánh class bằng log score:

$$
\log P(c)+\sum_{j=1}^{d}\log P(x_j\mid c).
$$

Giả định độc lập có điều kiện thường sai — chẳng hạn từ ngữ trong câu phụ thuộc nhau — nhưng classification vẫn có thể tốt, nhất là với feature text thưa. Tính trong log-space biến phép nhân thành phép cộng và tránh underflow số.

### Multinomial Naive Bayes và smoothing

Với word-count feature, ước lượng xác suất từ trong từng class bằng count. Nếu không smoothing, từ chưa xuất hiện trong class $c$ có xác suất 0 và làm toàn bộ tích likelihood bằng 0. Additive smoothing với $\alpha>0$ cho:

$$
P(\text{word }j\mid c)=\frac{N_{cj}+\alpha}{N_c+\alpha V},
$$

trong đó $N_{cj}$ là count của từ j trong class c, $N_c$ là tổng token của class đó, còn V là kích thước vocabulary. Smoothing strength là hyperparameter. Gaussian Naive Bayes giả định mỗi feature liên tục có Gaussian riêng theo class; Bernoulli Naive Bayes mô hình hóa sự hiện diện feature nhị phân.

## 6. So sánh model theo tác vụ

Với dữ liệu dạng bảng, so baseline hằng hoặc tuyến tính với tree nhỏ và tree ensemble có regularization. Với text count, Naive Bayes có thể rất nhanh và cạnh tranh. Dùng cùng data split, preprocessing rule và metric cho mọi ứng viên; kiểm tra lỗi ở subgroup quan trọng. Model đơn giản chỉ có giá trị nếu performance và vận hành phù hợp use case.

## Bài luyện tập kèm đáp án

1. Entropy của node thuần hai class là bao nhiêu? **Đáp án:** 0.
2. Vì sao impurity child được tính trọng số theo kích thước? **Đáp án:** child nhỏ thuần không nên được tính như thể toàn node đã thuần.
3. Vì sao Naive Bayes dùng log probability? **Đáp án:** đổi tích thành tổng và tránh underflow số.
4. Additive smoothing ngăn điều gì? **Đáp án:** cặp feature/class chưa từng thấy làm toàn bộ class likelihood bằng 0.
5. Ensemble tree nào chủ yếu giảm variance bằng cách lấy trung bình các tree được random hóa? **Đáp án:** random forest.
