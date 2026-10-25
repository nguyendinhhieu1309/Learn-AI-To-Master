# Linear regression và gradient descent

[Xem bài gốc bằng tiếng Anh](../../../../curriculum/04-machine-learning/07-linear-regression-and-gradient-descent.md)

Linear regression là ví dụ ML hoàn chỉnh đầu tiên: có representation, loss, phương pháp optimization, giả định, chẩn đoán và lộ trình từ tự tính đến dùng thư viện.

## Mục tiêu học tập

- Viết linear model dạng scalar và vector hóa, kiểm tra shape.
- Suy ra squared-error loss và gradient.
- Giải least squares bằng routine đại số tuyến tính số.
- Giải thích khi nào gradient descent hữu ích và learning rate hoạt động ra sao.
- Chẩn đoán pattern residual và dùng regularization có trách nhiệm.

## 1. Bài toán dự đoán

Giả sử có $n$ ví dụ. Mỗi hàng $x_i\in\mathbb{R}^{d}$ chứa $d$ feature, còn $y_i\in\mathbb{R}$ là target. Linear model dự đoán:

$$
\hat y_i=w^Tx_i+b,
$$

trong đó $w\in\mathbb{R}^{d}$ chứa coefficient của từng feature và $b$ là intercept. Coefficient $w_j$ là thay đổi prediction của model khi feature $j$ tăng một đơn vị, **giữ các feature khác cố định**. Đây là mô tả fitted model, không phải kết luận nhân quả.

Với batch theo row, đặt ví dụ vào $X\in\mathbb{R}^{n\times d}$:

$$
\hat y=Xw+b\mathbf{1},
\qquad X:(n,d),\;w:(d,),\;\hat y:(n,).
$$

Nếu gộp intercept vào matrix, thêm cột 1 vào $X$ và nối $b$ vào $w$. Chọn một quy ước rồi dùng nhất quán.

## 2. Loss: prediction sai bao nhiêu?

Gọi residual $r_i=\hat y_i-y_i$. Mean squared error:

$$
\operatorname{MSE}=\frac{1}{n}\sum_{i=1}^{n}r_i^2.
$$

Để đạo hàm gọn, nhiều phép suy ra tối thiểu hóa nửa mean squared error:

$$
J(w,b)=\frac{1}{2n}\sum_{i=1}^{n}(w^Tx_i+b-y_i)^2
=\frac{1}{2n}\|Xw+b\mathbf{1}-y\|_2^2.
$$

Hệ số $\frac12$ không đổi bộ tham số tối thiểu hóa loss. Bình phương làm residual lớn bị phạt nhiều hơn và tạo objective trơn, nhưng nhạy với outlier. Kiểm tra residual plot và cân nhắc MAE hoặc Huber loss nếu dữ liệu phù hợp objective robust.

## 3. Suy ra gradient

Với một ví dụ, $\ell=\tfrac12(wx+b-y)^2$. Prediction bên trong có đạo hàm $\partial(wx+b-y)/\partial w=x$ và $\partial(wx+b-y)/\partial b=1$. Theo chain rule:

$$
\frac{\partial\ell}{\partial w}=(wx+b-y)x,
\qquad
\frac{\partial\ell}{\partial b}=wx+b-y.
$$

Lấy trung bình trên dataset:

$$
\nabla_wJ=\frac{1}{n}X^T(Xw+b\mathbf{1}-y),
\qquad
\frac{\partial J}{\partial b}=\frac{1}{n}\mathbf{1}^T(Xw+b\mathbf{1}-y).
$$

Weight gradient có $d$ phần tử vì mỗi feature cần một hướng cập nhật. Kích thước matrix cũng xác nhận:

$$
(d\times n)(n\times1)=(d\times1).
$$

### Ví dụ tính một lần cập nhật

Dùng một ví dụ $x=2,y=5$, khởi tạo $w=1,b=0$. Prediction bằng 2, residual $\hat y-y=-3$ và half-squared loss bằng 4.5. Gradient là $(-6,-3)$. Với learning rate $\eta=0.1$:

$$
w_{\mathrm{new}}=1-0.1(-6)=1.6,
\qquad b_{\mathrm{new}}=0-0.1(-3)=0.3.
$$

Prediction sau cập nhật là 3.5, half-squared loss bằng 1.125. Bước này đã đi theo hướng làm giảm loss.

## 4. Gradient descent

Gradient descent lặp cập nhật:

$$
w\leftarrow w-\eta\nabla_wJ,
\qquad b\leftarrow b-\eta\frac{\partial J}{\partial b}.
$$

Learning rate $\eta$ kiểm soát kích thước bước. Rate quá lớn có thể làm loss tăng hoặc dao động; rate quá nhỏ khiến training rất chậm. Vẽ objective theo số lần cập nhật và tune dựa trên validation, không dựa trên final test set.

- **Batch gradient descent:** mỗi lần cập nhật dùng toàn bộ training example.
- **Stochastic gradient descent:** mỗi lần dùng một example; cập nhật nhiễu nhưng thường xuyên.
- **Mini-batch gradient descent:** lấy trung bình gradient trên batch nhỏ, là cách dung hòa phổ biến với phần cứng.
- **Momentum/Adam:** dùng thông tin gradient gần đây để làm trơn hoặc scale bước cập nhật. Chúng không loại bỏ nhu cầu kiểm tra dữ liệu, shape hay learning rate.

Standardize feature bằng thống kê fit từ training set thường cải thiện optimization. Đặt scaler trong training pipeline, sau đó áp dụng cùng transform đã lưu lên validation, test và production.

## 5. Nghiệm least-squares dạng đóng

Thêm cột intercept để tạo $\bar X=[\mathbf{1},X]$ và gộp coefficient vào $\theta=[b,w^T]^T$. Objective là $\|\bar X\theta-y\|_2^2$. Lấy đạo hàm và cho gradient bằng 0 thu được normal equation:

$$
\bar X^T\bar X\theta=\bar X^Ty.
$$

Nếu matrix có full column rank, có thể viết $\hat\theta=(\bar X^T\bar X)^{-1}\bar X^Ty$. Trong code, ưu tiên numpy.linalg.lstsq hoặc solver dựa trên QR/SVD thay vì tính inverse trực tiếp. Nếu các cột phụ thuộc, nghiệm có thể không duy nhất; least-squares solver trả về nghiệm ổn định theo quy ước đã tài liệu hóa.

### Ví dụ fit ba điểm

Cho $(x,y)=\{(0,1),(1,3),(2,5)\}$, các điểm nằm chính xác trên $y=2x+1$. Vì vậy $w=2,b=1$ và mọi residual đều bằng 0. Nếu label quan sát là 1, 2.8, 5.2 thì least squares sẽ cân bằng residual; đường fit thường không đi qua mọi điểm. Đánh giá model bằng held-out data và cấu trúc residual, không chỉ độ khớp với training point.

## 6. Feature engineering và giới hạn model

“Linear” nghĩa là tuyến tính theo coefficient học được. Model có thể mô tả đường cong sau khi biến đổi input, ví dụ:

$$
\phi(x)=[x, x^2, \sin x],
\qquad \hat y=w^T\phi(x)+b.
$$

Model vẫn tuyến tính theo $w$, dù quan hệ với $x$ ban đầu là đường cong. Feature cần có lý do và được kiểm tra overfit. Feature tương quan hoặc dư thừa có thể làm coefficient riêng lẻ bất ổn dù prediction tương tự.

## 7. Regularization

Ridge regression thêm penalty $L_2$, thường không áp dụng lên intercept:

$$
J_{\mathrm{ridge}}(w,b)=\operatorname{MSE}(w,b)+\lambda\|w\|_2^2.
$$

$\lambda$ lớn hơn làm coefficient co lại nhiều hơn. Lasso dùng $\lambda\|w\|_1$ và có thể đưa một số coefficient về đúng 0. Regularization có thể cải thiện generalization khi model chưa regularize quá nhạy, nhưng hãy tune $\lambda$ trên validation và scale feature vì penalty phụ thuộc đơn vị feature.

## 8. Chẩn đoán residual và giả định

Vẽ residual $y-\hat y$ theo prediction, từng feature, thời gian và group liên quan. Đường cong có thể gợi ý thiếu nonlinear term; residual spread tăng có thể báo phương sai không hằng; pattern theo group có thể gợi ý thiếu cấu trúc hoặc split sai. Muốn suy luận coefficient cần thêm giả định; với dự đoán, held-out performance và tính đại diện dữ liệu là trọng tâm.

## 9. Cài đặt Python tối thiểu

~~~python
import numpy as np

X = np.array([[0.0], [1.0], [2.0]])  # shape (examples, features)
y = np.array([1.0, 3.0, 5.0])       # shape (examples,)
X_with_bias = np.c_[np.ones(len(X)), X]

theta, residuals, rank, singular_values = np.linalg.lstsq(
    X_with_bias, y, rcond=None
)
b, w = theta
print(f"bias={b:.3f}, weight={w:.3f}")
~~~

Trong workflow production, đặt preprocessing và estimator trong cùng scikit-learn pipeline rồi đánh giá bằng split phù hợp cách ví dụ tương lai sẽ đến.

## Bài luyện tập kèm đáp án

1. Với $\hat y=wx+b$, tham số nào làm mọi prediction thay đổi cùng một lượng? **Đáp án:** bias $b$.
2. Với $x=3$, $y=4$, $w=1$, $b=0$, prediction và half-squared loss của một ví dụ là bao nhiêu? **Đáp án:** 3 và 0.5.
3. Vì sao inverse trực tiếp normal equation có thể không đáng tin? **Đáp án:** rank deficiency hoặc conditioning kém khiến inverse không tồn tại hay bất ổn số; dùng QR/SVD để giải least squares.
4. Residual plot training có dạng chữ U. Bước tiếp theo hợp lý là gì? **Đáp án:** xem xét nonlinear feature có cơ sở hoặc model phù hợp hơn, rồi đánh giá trên held-out data hợp lệ.

## Lỗi thường gặp

- Quên intercept hoặc preprocessing train và inference khác nhau.
- Nhầm training loss thấp với dự đoán tương lai tốt.
- Dùng inverse tường minh của $X^TX$ trong production.
- Xem coefficient là tác động nhân quả.
- Chọn feature, regularization hoặc metric bằng test set.
