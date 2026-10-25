# Logistic regression, perceptron và softmax

[Xem bài gốc bằng tiếng Anh](../../../../curriculum/04-machine-learning/08-logistic-regression-perceptron-and-softmax.md)

Các linear classifier này cùng tạo score $w^Tx+b$ nhưng chuyển score thành quyết định theo cách khác nhau. Hiểu rõ khác biệt giúp làm sáng tỏ loss, output và failure mode.

## Mục tiêu học tập

- Phân biệt linear score, xác suất và class dự đoán.
- Suy ra binary logistic model và binary cross-entropy.
- Giải thích perceptron khác logistic regression thế nào.
- Mở rộng binary classification sang class loại trừ lẫn nhau bằng softmax.
- Chọn threshold bằng validation data và chi phí lỗi.

## 1. Score, xác suất và quyết định

Với binary input $x$, linear score là $z=w^Tx+b$. Score có thể là mọi số thực nên tự nó không phải xác suất. Logistic regression đưa score qua sigmoid:

$$
\sigma(z)=\frac{1}{1+e^{-z}},
\qquad
p=P(y=1\mid x)=\sigma(w^Tx+b).
$$

Xác suất bổ sung là $P(y=0\mid x)=1-p$. Threshold $t$ đổi xác suất thành quyết định:

$$
\hat y=\begin{cases}1 & p\ge t,\\0 & p<t.\end{cases}
$$

Mặc định phổ biến $t=0.5$ tương ứng với chi phí hai loại lỗi bằng nhau và xác suất calibrated, nhưng chi phí kinh doanh, an toàn hay y tế có thể đòi threshold khác. Chọn bằng validation data và hậu quả dự kiến của false positive/false negative.

## 2. Log-odds tuyến tính

Lấy log odds:

$$
\log\frac{p}{1-p}=w^Tx+b.
$$

Mỗi coefficient làm log-odds dịch một lượng cố định khi feature tăng một đơn vị và các feature khác giữ nguyên. Mức thay đổi xác suất không cố định vì sigmoid là đường cong.

## 3. Binary cross-entropy và gradient

Với $y\in\{0,1\}$, Bernoulli likelihood là $p^y(1-p)^{1-y}$. Negative log-likelihood của nó là binary cross-entropy:

$$
\ell(p,y)=-\left[y\log p+(1-y)\log(1-p)\right].
$$

Với $n$ ví dụ, lấy trung bình loss. Gọi logits $z=Xw+b\mathbf{1}$ và probability $p=\sigma(z)$, gradient có dạng gọn:

$$
\nabla_w J=\frac{1}{n}X^T(p-y),
\qquad
\frac{\partial J}{\partial b}=\frac{1}{n}\mathbf{1}^T(p-y).
$$

Residual ở đây là xác suất dự đoán trừ target. Nếu ví dụ positive đang có xác suất quá thấp, $p-y<0$ và gradient descent sẽ cập nhật tham số để tăng score của nó.

### Tự kiểm tra một quan sát

Cho $x=2,y=1,w=0,b=0$. Khi đó $z=0,p=0.5$, loss $-\log(0.5)\approx0.693$. Gradient là $\partial J/\partial w=(p-y)x=-1$ và $\partial J/\partial b=-0.5$. Với learning rate 0.1, một bước cập nhật thành $w=0.1,b=0.05$, logit của ví dụ positive tăng từ 0 lên 0.25.

## 4. Vì sao không dùng perceptron update?

Perceptron dùng quyết định dấu cứng, thường là $\hat y=\operatorname{sign}(w^Tx+b)$ với nhãn $y\in\{-1,+1\}$. Khi phân loại sai một ví dụ, cập nhật cổ điển là:

$$
w\leftarrow w+\eta yx,
\qquad b\leftarrow b+\eta y.
$$

Perceptron tập trung sửa lỗi phân loại và không tạo xác suất calibrated. Logistic regression dùng objective xác suất trơn; nó cập nhật cả ví dụ phân loại đúng nếu probability còn kém. Cả hai có decision boundary tuyến tính trừ khi biến đổi feature.

## 5. Multiclass softmax regression

Với $K$ class loại trừ lẫn nhau, mỗi ví dụ có $K$ logit $z_k=w_k^Tx+b_k$. Softmax đổi chúng thành phân phối xác suất:

$$
p_k=\frac{e^{z_k}}{\sum_{j=1}^{K}e^{z_j}},
\qquad \sum_{k=1}^{K}p_k=1.
$$

Với one-hot label $y_k$, categorical cross-entropy là:

$$
\ell=-\sum_{k=1}^{K}y_k\log p_k=-\log p_{\text{true class}}.
$$

Gradient theo logit là $\partial\ell/\partial z_k=p_k-y_k$. Implementation thường kết hợp log-softmax và cross-entropy để ổn định số. Nếu framework loss nhận raw logits, hãy truyền logits; áp dụng softmax hai lần làm thay đổi phép tính.

Softmax giả định mỗi ví dụ có một class đúng. Nếu nhiều label có thể đồng thời đúng, dùng sigmoid độc lập và binary cross-entropy cho từng label (multi-label classification), không dùng softmax ép tổng xác suất bằng một.

## 6. Class imbalance, threshold và calibration

Accuracy có thể trông cao khi một class chiếm đa số. Kiểm tra confusion matrix, precision, recall, PR curve và số lượng/chi phí lỗi thực tế. Class weighting hoặc resampling có thể giúp training nhưng thay đổi sampling có thể ảnh hưởng calibration xác suất. Nếu xác suất dùng để ra quyết định hoặc ước lượng risk, hãy đánh giá calibration trên validation đại diện (ví dụ reliability diagram hoặc Brier score).

Không chọn threshold bằng test label rồi báo hiệu năng trên chính test set đó. Để dành test cho đánh giá cuối một lần.

## 7. Mẫu Python

~~~python
from sklearn.linear_model import LogisticRegression
from sklearn.pipeline import make_pipeline
from sklearn.preprocessing import StandardScaler

model = make_pipeline(
    StandardScaler(),
    LogisticRegression(class_weight="balanced", max_iter=1000),
)
model.fit(X_train, y_train)
probability = model.predict_proba(X_valid)[:, 1]
~~~

Chỉ chọn class_weight nếu phù hợp objective; đây không phải cách sửa mọi trường hợp. Tune threshold quyết định trên X_valid, rồi đánh giá toàn bộ quy trình đã chọn một lần trên test split.

## Bài luyện tập kèm đáp án

1. Nếu $z=0$, $\sigma(z)$ bằng bao nhiêu? **Đáp án:** 0.5.
2. Nhãn thật là 1, xác suất dự đoán 0.8. Binary cross-entropy bằng bao nhiêu? **Đáp án:** $-\log(0.8)\approx0.223$.
3. Vì sao threshold 0.5 không nhất thiết tối đa recall? **Đáp án:** có thể cần threshold thấp hơn; ngưỡng đánh đổi false positive và false negative.
4. Softmax có biểu diễn được hai positive label cùng lúc không? **Đáp án:** không; xác suất class có tổng bằng một và biểu diễn các class loại trừ lẫn nhau.

## Lỗi thường gặp

- Gọi logits là xác suất.
- Báo metric phụ thuộc threshold mà không nêu threshold.
- Chỉ dùng accuracy trên dataset mất cân bằng.
- Áp dụng softmax trước hàm cross-entropy cần raw logits.
- Cho rằng score phân loại cao đồng nghĩa probability calibrated tốt.
