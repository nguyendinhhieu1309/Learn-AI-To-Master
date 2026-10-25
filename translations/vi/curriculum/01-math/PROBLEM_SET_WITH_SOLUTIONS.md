# Toán cho AI: Bộ bài tập kèm lời giải chi tiết

Workbook này giúp bạn biến công thức trong các bài Toán thành phép tính có thể tự kiểm tra bằng tay. Hãy thử giải từng bài trước khi xem lời giải. Luôn ghi kèm đơn vị, shape và giả định bên cạnh các con số.

[Xem bản tiếng Anh](../../../../curriculum/01-math/PROBLEM_SET_WITH_SOLUTIONS.md)

## Cách dùng workbook

Với mỗi bài:

1. Ghi các đại lượng đã biết và đại lượng cần tìm.
2. Nêu công thức cùng các giả định áp dụng.
3. Tự tính bằng tay.
4. Chỉ dùng một chương trình ngắn để kiểm tra sau khi đã hiểu rõ phép suy diễn.

## Bài tập

### 1. Tích vô hướng và độ tương đồng

Cho $a=(1,2)$ và $b=(2,-1)$.

1. Tính $a^\top b$.
2. Tính chuẩn Euclid của cả hai vector.
3. Kết quả cho biết gì về góc giữa hai vector?

### 2. Shape của ma trận

Cho $X\in\mathbb{R}^{3\times 2}$ gồm ba mẫu và hai feature, $w\in\mathbb{R}^{2}$ và $b\in\mathbb{R}$.

1. $Xw$ có shape nào?
2. $\hat y=Xw+b$ có shape nào?
3. Giải thích vì sao có thể cộng một bias vô hướng cho mọi hàng.

### 3. Một bước gradient descent

Cực tiểu hóa $f(w)=\frac{1}{2}(w-3)^2$. Bắt đầu ở $w_0=0$ với learning rate $\eta=0.2$.

1. Tính $f'(w)$.
2. Tính $w_1=w_0-\eta f'(w_0)$.
3. So sánh $f(w_0)$ và $f(w_1)$.

### 4. Tự tính gradient cho linear regression

Dùng hàm mục tiêu

$$
J(w,b)=\frac{1}{2m}\sum_{i=1}^{m}(wx_i+b-y_i)^2
$$

với hai mẫu $(x,y)=(0,1)$ và $(1,3)$. Bắt đầu tại $w=b=0$.

1. Tính prediction, error và loss.
2. Tính $\partial J/\partial w$ và $\partial J/\partial b$.
3. Thực hiện một bước gradient với $\eta=0.1$.

### 5. Regularization L2

Giả sử $J_{\mathrm{reg}}(w)=J(w)+\frac{\lambda}{2}w^2$.

1. Lấy đạo hàm của penalty.
2. Nếu $J'(w)=-1.5$, $w=2$ và $\lambda=0.1$, gradient sau regularization là bao nhiêu?
3. Vì sao thường không áp dụng penalty lên intercept?

### 6. Bayes' rule và sự kiện hiếm

Một tình trạng ảnh hưởng đến 1% dân số. Xét nghiệm có sensitivity 90% và false-positive rate 5%. Một người nhận kết quả dương tính.

1. Tính xác suất xét nghiệm cho kết quả dương tính.
2. Tính xác suất người đó mắc tình trạng này khi đã biết kết quả dương tính.
3. Giải thích vì sao “sensitivity 90%” không có nghĩa là kết quả dương tính kéo theo 90% khả năng mắc bệnh.

### 7. Mean và variance

Với các quan sát $1,3,5$, hãy tính mean, population variance và sample variance. Nêu mẫu số được dùng cho mỗi variance.

### 8. Standard error và confidence interval xấp xỉ

Một sample có $n=16$, mean $\bar{x}=12$ và sample standard deviation $s=4$. Dùng hệ số normal 1.96 của mẫu lớn để xấp xỉ:

1. Ước lượng standard error của mean.
2. Tính khoảng 95% xấp xỉ.
3. Nêu một lý do khiến phép xấp xỉ này có thể không đáng tin.

### 9. Bernoulli MLE và MAP

Trong 10 phép thử độc lập có 7 lần thành công. Gọi $p$ là xác suất thành công.

1. Tìm maximum-likelihood estimate.
2. Dùng prior $\operatorname{Beta}(2,2)$ và tìm posterior.
3. Tìm posterior mode và so sánh với MLE.

### 10. Cross-entropy

Với một mẫu nhị phân có nhãn thật $y=1$ và xác suất dự đoán $p=0.8$, hãy tính binary cross-entropy. Điều gì xảy ra với loss khi $p$ tiến gần về 0?

### 11. PCA trong trường hợp đơn giản

Xét hai quan sát hai chiều đã được center: $(1,0)$ và $(-1,0)$. Dùng population covariance (chia cho $n$):

1. Tính ma trận covariance.
2. Tìm eigenvalue và một principal eigenvector.
3. Chiếu dữ liệu lên principal component thứ nhất làm mất thông tin nào?

### 12. Kiểm tra gradient bằng sai phân hữu hạn

Với $f(w)=w^2$ tại $w=2$, hãy xấp xỉ đạo hàm bằng central difference với $h=0.001$:

$$
\frac{f(w+h)-f(w-h)}{2h}.
$$

So sánh với đạo hàm giải tích.

### 13. Tính lồi từ Hessian

Với $f(x,y)=x^2+3y^2$, hãy tính gradient và Hessian. Hàm này có convex không? Có strictly convex không?

### 14. Entropy và độ bất định

Một biến nhị phân công bằng có xác suất $(0.5,0.5)$. Hãy tính entropy theo bit. Sau đó tính entropy với xác suất $(0.9,0.1)$ và giải thích sự khác nhau.

## Lời giải chi tiết

### Lời giải 1

$$
a^\top b=1(2)+2(-1)=0,\qquad
\|a\|_2=\sqrt{1^2+2^2}=\sqrt{5},\qquad
\|b\|_2=\sqrt{2^2+(-1)^2}=\sqrt{5}.
$$

Cosine similarity bằng $0/(\sqrt5\sqrt5)=0$, vì vậy hai vector vuông góc.

### Lời giải 2

$Xw$ có shape $(3\times2)(2\times1)=3\times1$. Bias là một scalar được broadcast vào cả ba prediction, nên $\hat y$ cũng có shape $(3,1)$. Trong code, hãy kiểm tra rõ batch gồm ba mẫu vẫn nằm trên axis đầu tiên; transpose nhầm có thể tạo ra shape trông hợp lệ nhưng sai.

### Lời giải 3

$$
f'(w)=w-3,\qquad f'(0)=-3,\qquad
w_1=0-0.2(-3)=0.6.
$$

Loss ban đầu là $\frac12(0-3)^2=4.5$. Loss mới là $\frac12(0.6-3)^2=2.88$. Bước cập nhật này làm giảm objective.

### Lời giải 4

Tại $(w,b)=(0,0)$, cả hai prediction đều bằng 0 và error $\hat y-y$ lần lượt là $-1$ và $-3$. Do đó:

$$
J=\frac{1}{4}(1+9)=2.5.
$$

Với objective này:

$$
\frac{\partial J}{\partial w}=\frac1m\sum_i(\hat y_i-y_i)x_i=-1.5,
\qquad
\frac{\partial J}{\partial b}=\frac1m\sum_i(\hat y_i-y_i)=-2.
$$

Một bước cập nhật cho $w_1=0-0.1(-1.5)=0.15$ và $b_1=0-0.1(-2)=0.2$.

### Lời giải 5

Đạo hàm của $\frac{\lambda}{2}w^2$ là $\lambda w$. Gradient có regularization bằng $-1.5+0.1(2)=-1.3$. Nếu phạt intercept thì toàn bộ prediction có thể bị đẩy lệch khỏi baseline tự nhiên của target; phần lớn quy ước cho linear model regularize slope nhưng giữ intercept không bị phạt.

### Lời giải 6

Gọi $D$ là “mắc tình trạng” và $+$ là “xét nghiệm dương tính”.

$$
P(+)=P(+\mid D)P(D)+P(+\mid\neg D)P(\neg D)
=0.9(0.01)+0.05(0.99)=0.0585.
$$

Khi đó:

$$
P(D\mid+)=\frac{P(+\mid D)P(D)}{P(+)}
=\frac{0.009}{0.0585}\approx0.154.
$$

Theo các giả định này, chỉ khoảng 15.4% kết quả dương tính thực sự mắc tình trạng. Vì tình trạng hiếm, số false positive từ nhóm người khỏe mạnh đông hơn nhiều có thể lớn so với số true positive.

### Lời giải 7

Mean bằng $3$. Bình phương độ lệch là $4,0,4$, tổng bằng $8$. Population variance là $8/3$. Sample variance là $8/(3-1)=4$. Công thức sample dùng $n-1$ vì mean được ước lượng từ chính sample đó.

### Lời giải 8

$$
\operatorname{SE}(\bar{x})\approx\frac{s}{\sqrt n}=\frac4{4}=1,
\qquad
12\pm1.96(1)=[10.04,13.96].
$$

Với chỉ 16 quan sát, phân phối lệch mạnh hoặc outlier lớn có thể khiến xấp xỉ normal kém chính xác. Khi ước lượng mean mà variance chưa biết, thường dùng hệ số $t$, đặc biệt với sample nhỏ.

### Lời giải 9

Bernoulli likelihood tỷ lệ với $p^7(1-p)^3$, vì vậy $\hat p_{\mathrm{MLE}}=7/10=0.7$. Tính liên hợp Beta cho posterior $\operatorname{Beta}(2+7,2+3)=\operatorname{Beta}(9,5)$. Mode của posterior là:

$$
\hat p_{\mathrm{MAP}}=\frac{9-1}{9+5-2}=\frac8{12}\approx0.667.
$$

Prior kéo ước lượng về phía $0.5$. MAP là mode của posterior; posterior mean sẽ là $9/14\approx0.643$.

### Lời giải 10

$$
\ell(y,p)=-y\log p-(1-y)\log(1-p)=-\log(0.8)\approx0.223.
$$

Khi nhãn thật là 1 còn $p$ tiến gần 0, loss tăng không giới hạn. Logarithm phạt mạnh các dự đoán sai nhưng quá tự tin.

### Lời giải 11

Mean đã bằng 0. Population covariance là:

$$
\Sigma=\frac12\left(
\begin{bmatrix}1\\0\end{bmatrix}\begin{bmatrix}1&0\end{bmatrix}
+
\begin{bmatrix}-1\\0\end{bmatrix}\begin{bmatrix}-1&0\end{bmatrix}
\right)
=\begin{bmatrix}1&0\\0&0\end{bmatrix}.
$$

Eigenvalue là 1 và 0; eigenvector ứng với eigenvalue lớn nhất là $(1,0)$. Chiếu lên nó không làm mất độ biến thiên nào trong ví dụ này vì mọi điểm đều có tọa độ thứ hai bằng 0. Với dữ liệu thực tế, các component bị bỏ có thể chứa thông tin liên quan đến tác vụ dù variance nhỏ.

### Lời giải 12

$$
\frac{(2.001)^2-(1.999)^2}{0.002}
=\frac{4.004001-3.996001}{0.002}=4.
$$

Đạo hàm giải tích là $2w=4$. Sai phân hữu hạn thường chỉ là xấp xỉ; cần chọn step size để cân bằng truncation error và hiện tượng triệt tiêu do floating-point.

### Lời giải 13

$$
\nabla f(x,y)=\begin{bmatrix}2x\\6y\end{bmatrix},\qquad
\nabla^2 f(x,y)=\begin{bmatrix}2&0\\0&6\end{bmatrix}.
$$

Hai eigenvalue của Hessian đều dương, nên hàm strictly convex và có global minimum duy nhất tại $(0,0)$.

### Lời giải 14

Dùng $H(p)=-\sum_i p_i\log_2 p_i$:

$$
H(0.5,0.5)=1\text{ bit},
\qquad
H(0.9,0.1)=-0.9\log_2(0.9)-0.1\log_2(0.1)\approx0.469\text{ bit}.
$$

Biến thứ hai ít bất định hơn vì một kết quả có khả năng xảy ra cao hơn hẳn kết quả còn lại.

## Kiểm tra mức độ thành thạo

Bạn sẵn sàng chuyển tiếp khi có thể tự suy ra gradient của regression, giải thích ảnh hưởng của base rate trong Bayes' rule, phân biệt MLE với MAP và kiểm tra gradient bằng số mà không chép lời giải.
