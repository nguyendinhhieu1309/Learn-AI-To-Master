# Giải tích, gradient và tối ưu hóa

> Bản tiếng Anh: [Calculus, Gradients, and Optimization](../../../../curriculum/01-math/02-calculus-and-optimization.md)

## Mục tiêu học tập

Hiểu đạo hàm là tốc độ thay đổi cục bộ, gradient là hướng thay đổi và gradient descent là các bước lặp để giảm loss.

## 1. Đạo hàm: một giá trị thay đổi nhanh đến đâu?

Với hàm $f(x)$, đạo hàm $f'(x)$ cho biết độ dốc gần $x$. Hàm đường thẳng $f(x)=3x+1$ có đạo hàm 3 ở mọi điểm: nếu tăng $x$ một lượng nhỏ, $f$ tăng xấp xỉ ba lần lượng đó. Với $f(x)=x^2$, đạo hàm là $2x$; tại $x=3$, độ dốc bằng 6.

Đạo hàm là đại lượng cục bộ. Với bước nhảy lớn, độ dốc có thể thay đổi nên xấp xỉ tuyến tính có thể không còn tốt.

## 2. Đạo hàm riêng và gradient

Mô hình có nhiều tham số. Đạo hàm riêng thay đổi một tham số trong khi giữ các tham số khác cố định. **Gradient** gom tất cả đạo hàm riêng:

$$
\nabla_\theta J(\theta) = \left[\frac{\partial J}{\partial \theta_1}, ..., \frac{\partial J}{\partial \theta_d}\right].
$$

Gradient chỉ hướng làm $J$ tăng cục bộ. Muốn giảm loss, ta đi theo hướng ngược lại.

## 3. Tính gradient descent bằng tay

Tối thiểu hóa $J(w)=(w-3)^2$. Đạo hàm là $2(w-3)$. Bắt đầu tại $w=0$ với learning rate $\\eta=0.1$, cập nhật:

$$
w_{\mathrm{new}}=w-\eta J'(w)=0-0.1(-6)=0.6.
$$

Lặp lại sẽ đưa $w$ đến gần 3 và làm loss giảm. Learning rate quá nhỏ khiến tiến triển chậm. Nếu quá lớn, các bước có thể nhảy qua điểm cực tiểu hoặc phân kỳ.

## 4. Chain rule và neural network

Nếu $y=f(g(x))$ thì:

$$
\frac{dy}{dx}=f'(g(x))g'(x).
$$

Chain rule cho phép tính đầu ra thay đổi ra sao qua một chuỗi layer. Backpropagation áp dụng quy tắc này hiệu quả cho mọi tham số trong neural network.

Chain rule không đảm bảo gradient luôn truyền tốt: phép nhân lặp lại có thể làm gradient rất nhỏ (vanishing gradient) hoặc rất lớn (exploding gradient). Khởi tạo, kiến trúc, normalization và cấu hình optimizer có thể giúp xử lý.

## 5. Loss và regularization

**Loss** đo sai số dự đoán. Mean squared error là:

$$
\operatorname{MSE}=\frac{1}{n}\sum_{i=1}^{n}(y_i-\hat y_i)^2.
$$

Trong classification, cross-entropy phạt nặng khi xác suất gán cho lớp đúng thấp. Optimizer tối thiểu hóa loss trung bình trên tập train.

Regularization cộng thêm một khoản phạt, ví dụ:

$$
J_{\mathrm{total}}=J_{\mathrm{data}}+\lambda\|w\|_2^2.
$$

Nó ngăn trọng số tăng quá lớn. Hệ số phạt $\\lambda$ là hyperparameter; hãy điều chỉnh bằng tập validation.

## 6. Vòng lặp tối ưu hóa thực tế

1. Tạo dự đoán ở forward pass.
2. So sánh dự đoán với label để tính loss.
3. Tính gradient của loss theo các tham số.
4. Dùng optimizer cập nhật tham số.
5. Lặp qua các batch; đo riêng hiệu năng trên validation.

Quan sát đường cong loss. Training loss giảm trong khi validation loss tăng là dấu hiệu overfitting. Nếu cả hai đều cao, có thể mô hình quá đơn giản, đầu vào kém, label sai hoặc tối ưu hóa có vấn đề.

## Tự kiểm tra

Với $J(w)=(w-2)^2$, tại $w=5$, gradient bằng 6. Với learning rate $\\eta=0.1$, giá trị tiếp theo là $5-0.1(6)=4.4$. Kiểm tra xem $w$ đã dịch về phía 2 chưa. Phép kiểm tra một bước này hữu ích để phát hiện nhầm dấu.

## Lỗi thường gặp

- Cập nhật theo hướng gradient khi mục tiêu là tối thiểu hóa.
- Cho rằng training loss thấp hơn luôn có nghĩa mô hình tổng quát hóa tốt hơn.
- Thay learning rate, mô hình và dữ liệu cùng lúc khiến khó tìm nguyên nhân.
- Quên rằng gradient phụ thuộc batch dữ liệu và giá trị tham số hiện tại.

## Suy ra gradient cho linear regression

Phần trước giới thiệu ý tưởng gradient descent. Phần này suy ra cập nhật của một mô hình thật để tránh học thuộc quy tắc mà không hiểu ký hiệu và dấu.

Giả sử có $n$ ví dụ và một feature. Ví dụ $i$ có đầu vào $x_i$, target $y_i$, dự đoán $\\hat y_i=wx_i+b$ và residual $r_i=\\hat y_i-y_i$. Dùng half mean squared error:

$$
J(w,b)=\frac{1}{2n}\sum_{i=1}^{n}(wx_i+b-y_i)^2.
$$

Hệ số $1/2$ triệt tiêu hệ số 2 xuất hiện khi lấy đạo hàm bình phương. Áp dụng chain rule cho một thành phần:

$$
\frac{\partial J}{\partial w}
=\frac{1}{n}\sum_{i=1}^{n}(wx_i+b-y_i)x_i,
\qquad
\frac{\partial J}{\partial b}
=\frac{1}{n}\sum_{i=1}^{n}(wx_i+b-y_i).
$$

Vì sao gradient theo trọng số có $x_i$? Tăng $w$ làm dự đoán thay đổi $x_i$ đơn vị; tăng $b$ làm mọi dự đoán thay đổi một đơn vị.

### Một bước cập nhật có lời giải

Cho $x=2$, $y=5$, $w=1$, $b=0$. Dự đoán là $\\hat y=2$ và half-squared loss là $J=\\tfrac12(2-5)^2=4.5$. Gradient:

$$
\frac{\partial J}{\partial w}=(2-5)\cdot2=-6,
\qquad
\frac{\partial J}{\partial b}=2-5=-3.
$$

Với learning rate $\\eta=0.1$, cập nhật cả hai tham số bằng gradient **cũ**:

$$
w_{\mathrm{new}}=1-0.1(-6)=1.6,
\qquad
b_{\mathrm{new}}=0-0.1(-3)=0.3.
$$

Dự đoán mới là $2(1.6)+0.3=3.5$ và loss mới là $\\tfrac12(3.5-5)^2=1.125$. Loss giảm, đó là dấu hiệu hợp lý. Một ví dụ không đủ để kết luận cập nhật tổng quát hóa sang dữ liệu mới.

## Từ ví dụ vô hướng đến gradient ma trận

Với $n$ ví dụ và $d$ feature, đặt mỗi ví dụ trên một hàng của $X\\in\\mathbb{R}^{n\\times d}$, dùng $w\\in\\mathbb{R}^{d}$ và $y\\in\\mathbb{R}^{n}$. Vector dự đoán và residual:

$$
\hat y=Xw+b\mathbf{1},
\qquad
r=Xw+b\mathbf{1}-y.
$$

Half mean squared error và gradient:

$$
J(w,b)=\frac{1}{2n}r^Tr,
\qquad
\nabla_w J=\frac{1}{n}X^Tr,
\qquad
\frac{\partial J}{\partial b}=\frac{1}{n}\mathbf{1}^Tr.
$$

Kiểm tra kích thước trước khi nhân:

$$
(n\times d)^T(n\times1)=(d\times n)(n\times1)=(d\times1),
$$

shape kết quả khớp với số trọng số. Với nhiều output, đạo hàm có shape giống ma trận weight. Kiểm tra shape là cách đơn giản để bắt nhiều lỗi giải tích và code.

## Các quy tắc đạo hàm cần biết

Với hằng số $a,b$ và hàm khả vi $f,g$:

$$
\frac{d}{dx}c=0,\quad
\frac{d}{dx}x^k=kx^{k-1},\quad
\frac{d}{dx}e^x=e^x,\quad
\frac{d}{dx}\ln x=\frac{1}{x},
$$

$$
\frac{d}{dx}[af(x)+bg(x)]=af'(x)+bg'(x),\quad
\frac{d}{dx}f(g(x))=f'(g(x))g'(x).
$$

Đẳng thức cuối là chain rule. Với sigmoid $\\sigma(z)=1/(1+e^{-z})$:

$$
\sigma'(z)=\sigma(z)(1-\sigma(z)).
$$

Với hàm vector đầu ra, các đạo hàm tạo thành **Jacobian**: hàng ứng với output, cột ứng với input. Loss vô hướng theo vector tham số có gradient gồm một thành phần cho mỗi tham số. Hãy nêu quy ước đang dùng; phép chuyển vị khác nhau giữa quy ước gradient hàng và gradient cột.

## Kiểm tra gradient bằng số

Nếu chưa chắc gradient giải tích, hãy so sánh một thành phần với finite difference trung tâm:

$$
\frac{\partial J}{\partial w_j}\approx
\frac{J(w+\epsilon e_j)-J(w-\epsilon e_j)}{2\epsilon},
$$

trong đó $e_j$ chỉ làm thay đổi thành phần $j$ và $\\epsilon$ nhỏ. Trên bài toán rất nhỏ với double precision, hai giá trị giải tích và số phải gần nhau. Cách này giúp debug đạo hàm tự cài đặt; finite difference quá tốn kém và thiếu chính xác để huấn luyện neural network lớn.

## Điều gì có thể làm gradient descent thất bại?

- **Learning rate quá lớn:** bước cập nhật nhảy qua đáy và làm loss tăng. Giảm learning rate, quan sát loss theo bước.
- **Learning rate quá nhỏ:** loss giảm rất chậm. Thử tăng rate trên thí nghiệm nhỏ, an toàn.
- **Feature có scale rất khác nhau:** mặt loss có thể hẹp theo một hướng và rộng theo hướng khác. Fit scaler trên train rồi dùng lại cho validation và inference.
- **Gradient hoặc dữ liệu sai:** kiểm tra giá trị hữu hạn, label, tensor shape và khả năng overfit một batch nhỏ trước khi đổi kiến trúc.
- **Objective không lồi:** loss của neural network có vùng phẳng và saddle point; gradient bằng 0 không đảm bảo đó là cực tiểu toàn cục.

Batch gradient descent dùng toàn bộ ví dụ cho mỗi lần cập nhật. Stochastic gradient descent dùng một ví dụ; mini-batch gradient descent dùng batch nhỏ. Mini-batch đổi gradient nhiều nhiễu hơn để lấy các cập nhật thường xuyên và nhanh hơn. Momentum và optimizer thích nghi điều chỉnh lịch sử cập nhật hoặc scale từng tọa độ; chúng vẫn cần dữ liệu, gradient và learning rate hợp lý.

## Luyện tập có đáp án

1. Với $J(w)=\\tfrac12(3w-6)^2$, tính $dJ/dw$ tại $w=1$. **Đáp án:** $3w-6=-3$, nên $dJ/dw=(3w-6)\\cdot3=-9$.
2. Với $\\eta=0.1$, $w$ tiếp theo bằng bao nhiêu? **Đáp án:** $1-0.1(-9)=1.9$, gần cực tiểu $w=2$ hơn.
3. Nếu X có shape (100,8), r có shape (100,), shape của $X^Tr$ là gì? **Đáp án:** (8,), mỗi feature có một gradient.
4. Mục đích chính của numerical gradient check là gì? **Đáp án:** phát hiện lỗi cài đặt hoặc chain rule trên bài toán nhỏ.
