# Supervised learning và baseline hữu ích

[Xem bài gốc bằng tiếng Anh](../../../../curriculum/04-machine-learning/01-supervised-learning.md)

## Mục tiêu học tập

Định nghĩa bài toán input-output, xây baseline và hiểu hành vi cơ bản của các supervised model thường gặp.

## 1. Supervised learning là gì?

Ta có các ví dụ $(x_i,y_i)$, trong đó $x_i$ chứa input feature còn $y_i$ là target đã biết. Model học hàm $f(x)$ để dự đoán target cho ví dụ chưa gặp. Trong **regression**, target là số. Trong **classification**, target là category.

Hãy viết rõ bài toán trước khi code: Điều gì đã biết tại thời điểm dự đoán? Outcome nào quan trọng? Một lỗi gây tốn kém ra sao? Prediction sẽ được sử dụng thế nào?

## 2. Luôn thiết lập baseline

Baseline là mốc tham chiếu đơn giản. Với regression, dự đoán mean hoặc median của training set. Với classification, dự đoán class phổ biến nhất. Quy tắc dựa trên domain có thể là baseline tốt hơn. Nếu model phức tạp không thắng baseline hợp lý trên dữ liệu chưa đụng đến, hãy điều tra data và task trước khi tăng complexity.

## 3. Linear regression

Model tạo tổng có trọng số của feature:

$$
\hat y=w^Tx+b.
$$

Training thường tối thiểu hóa mean squared error:

$$
\operatorname{MSE}=\frac{1}{n}\sum_{i=1}^{n}(y_i-\hat y_i)^2.
$$

Coefficient mô tả mức thay đổi prediction khi feature tăng một đơn vị, **giữ các feature khác trong model cố định**. Feature tương quan có thể khiến cách diễn giải này bất ổn; coefficient cũng không tự động là quan hệ nhân quả.

Scale của feature ảnh hưởng đến optimization và regularization. Standardization biến đổi feature thành:

$$
x_{\mathrm{scaled}}=\frac{x-\mu_{\mathrm{train}}}{s_{\mathrm{train}}};
$$

tính $\mu_{\mathrm{train}}$ và $s_{\mathrm{train}}$ chỉ từ training data.

## 4. Logistic regression

Với binary classification, logistic regression đổi linear score thành xác suất:

$$
z=w^Tx+b,
\qquad p=P(y=1\mid x)=\sigma(z)=\frac{1}{1+e^{-z}}.
$$

Quyết định class mặc định thường dùng threshold 0.5, nhưng không phải lúc nào cũng đúng. Model tuyến tính theo feature dù xác suất đầu ra là phi tuyến.

## 5. Tree và ensemble

Decision tree liên tục chia feature space bằng câu hỏi như age < 30? Tree có thể mô hình hóa tương tác phi tuyến và ít cần scale feature, nhưng tree sâu có thể ghi nhớ training example. Random forest lấy trung bình nhiều tree được random hóa. Gradient-boosted tree lần lượt thêm tree để sửa lỗi trước đó. Ensemble có thể mạnh trên dữ liệu dạng bảng nhưng vẫn cần validation và error analysis cẩn thận.

## 6. Regularization và độ phức tạp model

Model quá đơn giản bị underfit; model quá linh hoạt có thể overfit. Regularization, early stopping, giới hạn độ sâu tree và thêm dữ liệu có thể hữu ích. Chọn setting bằng validation hoặc cross-validation, không dùng final test set.

## Ví dụ so sánh có hướng dẫn

Giả sử binary model dự đoán khách hàng có vỡ nợ khoản vay không. Bắt đầu với baseline majority-class. Sau đó so sánh logistic regression và tree ensemble trên cùng split. Giữ threshold cố định hoặc giải thích việc chọn threshold bằng validation data. Báo confusion count, precision/recall theo class chứ không chỉ accuracy.

Ví dụ baseline gọn sau dùng dataset đi kèm scikit-learn. Phần held-out không được dùng fit scaler hay classifier vì cả hai nằm trong pipeline.

~~~python
from sklearn.datasets import load_iris
from sklearn.linear_model import LogisticRegression
from sklearn.metrics import classification_report
from sklearn.model_selection import train_test_split
from sklearn.pipeline import make_pipeline
from sklearn.preprocessing import StandardScaler

X, y = load_iris(return_X_y=True)
X_train, X_test, y_train, y_test = train_test_split(
    X, y, test_size=0.25, random_state=7, stratify=y
)
model = make_pipeline(StandardScaler(), LogisticRegression(max_iter=1000))
model.fit(X_train, y_train)
predicted = model.predict(X_test)
print(classification_report(y_test, predicted))
~~~

Random seed cố định giúp chia lại giống nhau trong cùng toolchain; nó không cho biết performance dao động thế nào giữa các sample có thể có. Iris là ví dụ giảng dạy, không chứng minh score tương tự sẽ đạt được ở tác vụ production.

## Thực hành

Chọn dataset, ghi thông tin có sẵn tại thời điểm dự đoán và một baseline đơn giản trước khi training. So sánh linear model với tree-based model. Dùng lỗi trên held-out data để giải thích một điểm mạnh và một giới hạn của mỗi model.

## Lỗi thường gặp

- Gọi model là chính xác mà không so sánh với baseline.
- Diễn giải coefficient như tác động nhân quả.
- Scale bằng thống kê tính từ toàn bộ dataset.
- Chọn model tốt nhất dựa trên test sau nhiều lần thử nghiệm.
