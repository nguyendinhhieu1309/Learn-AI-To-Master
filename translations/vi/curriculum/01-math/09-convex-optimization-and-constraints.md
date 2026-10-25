# Tối ưu lồi và các ràng buộc

> Bản tiếng Anh: [Convex Optimization and Constraints](../../../../curriculum/01-math/09-convex-optimization-and-constraints.md)

Huấn luyện Machine Learning thường được viết thành bài toán tối ưu: chọn tham số để tối thiểu hóa loss nhưng vẫn đáp ứng các ràng buộc thực tế hoặc Toán học. Tính lồi giải thích khi nào nghiệm cục bộ cũng là nghiệm toàn cục và vì sao một số mô hình cổ điển dễ huấn luyện hơn neural network.

## Mục tiêu học tập

- Nhận biết tập lồi và hàm lồi qua định nghĩa và ví dụ.
- Liên hệ gradient, Hessian với cực tiểu cục bộ và độ cong.
- Giải thích vì sao least squares và logistic loss có nghiệm toàn cục, còn neural network thường không lồi.
- Viết objective có ràng buộc bằng Lagrangian và hiểu khái quát điều kiện KKT.
- Giải thích regularization và ràng buộc ảnh hưởng việc fit mô hình thế nào.

## 1. Tập lồi

Tập C lồi nếu đoạn thẳng nối hai điểm bất kỳ trong tập cũng nằm trọn trong tập. Với $x,y\in C$ và $\alpha\in[0,1]$:

$$
\alpha x+(1-\alpha)y\in C.
$$

Ví dụ: đoạn thẳng, đường thẳng, hình tròn đặc, half-space và tập ma trận positive semidefinite. Đường tròn chỉ có chu vi mà không có phần bên trong không phải tập lồi: dây cung giữa hai điểm biên đi qua phần bên trong bị loại trừ.

## 2. Hàm lồi

Hàm f lồi trên miền lồi nếu đồ thị của nó nằm dưới hoặc trên dây cung? Định nghĩa đúng là giá trị hàm tại điểm nội suy không vượt quá nội suy giá trị tại hai đầu:

$$
f(\alpha x+(1-\alpha)y)
\leq \alpha f(x)+(1-\alpha)f(y)
\quad\forall x,y,\;\alpha\in[0,1].
$$

Với hàm khả vi một chiều, đạo hàm bậc hai không âm ở mọi nơi là điều kiện đủ cho tính lồi. Với hàm khả vi hai lần theo vector, Hessian positive semidefinite ở mọi nơi là điều kiện đủ.

Ví dụ:

- $x^2$ lồi; mọi cực tiểu cục bộ cũng là cực tiểu toàn cục.
- $|x|$ lồi nhưng không khả vi tại 0.
- $e^x$ lồi.
- Hàm tuyến tính vừa lồi vừa lõm.
- $\sin x$ không lồi trên toàn trục thực vì độ cong đổi dấu.

Với hàm lồi khả vi, điểm có gradient bằng 0 là cực tiểu toàn cục. Vùng phẳng có thể chứa nhiều nghiệm tối ưu toàn cục. Lồi không nhất thiết đồng nghĩa nghiệm duy nhất.

## 3. Độ cong và Hessian

Với hàm vô hướng f(x), đạo hàm biểu diễn độ dốc, đạo hàm bậc hai biểu diễn độ cong. Với vector tham số $\theta$, Hessian gom đạo hàm bậc hai:

$$
H_{ij}=\frac{\partial^2J}{\partial\theta_i\partial\theta_j}.
$$

Hessian positive semidefinite nghĩa là tại chỗ không có hướng nào cong xuống; positive definite nghĩa là độ cong dương theo mọi hướng khác 0. Hessian condition number xấu có độ cong rất khác nhau giữa các hướng, làm gradient descent zig-zag nếu chọn step size không phù hợp.

## 4. Vì sao least-squares linear regression là convex?

Với design matrix X và target y, least-squares loss:

$$
J(w)=\frac12\|Xw-y\|_2^2.
$$

Gradient và Hessian:

$$
\nabla J(w)=X^T(Xw-y),
\qquad
\nabla^2J(w)=X^TX.
$$

Với mọi vector v:

$$
v^TX^TXv=\|Xv\|_2^2\geq0,
$$

nên Hessian positive semidefinite và objective lồi. Nếu X có full column rank, $X^TX$ positive definite và nghiệm duy nhất. Nếu các cột phụ thuộc, có thể có nhiều vector hệ số cho cùng dự đoán. Ridge thêm độ cong dương và có thể tạo nghiệm duy nhất khi $\lambda>0$.

Logistic regression với convex log-loss cũng lồi theo tham số tuyến tính. Tối ưu SVM hinge-loss cũng lồi. Với các mô hình này, optimizer đáng tin có thể tìm global optimum của objective đã nêu; tuy nhiên chất lượng dữ liệu và lựa chọn objective vẫn quyết định nghiệm đó có hữu ích không.

## 5. Vì sao loss của deep network thường không lồi?

Output neural network phụ thuộc tích và phép hợp thành của tham số nhiều layer. Cùng lúc thay đổi hai layer có thể tạo valley, saddle, đối xứng và nhiều cấu hình tham số tương đương. Loss thường không lồi, nên điểm dừng không được đảm bảo là cực tiểu toàn cục.

Điều đó không có nghĩa việc huấn luyện vô vọng. Mạng lớn thường có nhiều nghiệm với hành vi train hữu ích; khởi tạo, kiến trúc, dữ liệu, regularization và optimizer đều ảnh hưởng nghiệm đạt được. Validation performance và robustness quan trọng hơn bảo đảm lý thuyết rằng đã tìm ra cực tiểu toàn cục.

## 6. Tối ưu có ràng buộc và Lagrangian

Một số bài toán tối thiểu hóa objective với bất đẳng thức và đẳng thức:

$$
\min_\theta f(\theta)
\quad\text{subject to}\quad g_i(\theta)\leq0,\;h_j(\theta)=0.
$$

Lagrangian kết hợp objective và ràng buộc bằng multiplier:

$$
\mathcal{L}(\theta,\alpha,\nu)
=f(\theta)+\sum_i\alpha_i g_i(\theta)+\sum_j\nu_jh_j(\theta),
\qquad \alpha_i\geq0.
$$

Với bài toán lồi có điều kiện chính quy phù hợp, nghiệm thỏa điều kiện Karush–Kuhn–Tucker (KKT):

1. **Primal feasibility:** các ràng buộc ban đầu được thỏa mãn.
2. **Dual feasibility:** multiplier của bất đẳng thức thỏa $\alpha_i\geq0$.
3. **Stationarity:** gradient Lagrangian theo tham số bằng 0.
4. **Complementary slackness:** $\alpha_i g_i(\theta)=0$ cho mỗi bất đẳng thức.

Complementary slackness nói rằng ràng buộc chưa active có multiplier bằng 0; ràng buộc ảnh hưởng đến nghiệm có thể có multiplier dương.

Với hard-margin SVM, ràng buộc margin là $y_i(w^Tx_i+b)\geq1$. Đưa multiplier vào sẽ tạo dual problem phụ thuộc dot product giữa ví dụ. Thay dot product bằng kernel giúp có ranh giới SVM phi tuyến. Lúc đầu có thể bỏ qua suy ra dual đầy đủ; trực giác hình học và soft-margin được học trước.

## 7. Regularization như prior hoặc ràng buộc

Dạng có penalty và dạng có ràng buộc có liên hệ:

$$
\min_\theta L(\theta)+\lambda\|\theta\|_2^2
\qquad\leftrightarrow\qquad
\min_\theta L(\theta)\;\text{subject to}\;\|\theta\|_2\leq c.
$$

Tham số $\lambda$ hoặc bán kính c điều khiển đánh đổi giữa fit dữ liệu và giới hạn độ lớn tham số. Quan hệ chính xác phụ thuộc bài toán và việc ràng buộc có active hay không. Mục tiêu của regularization là cải thiện hành vi trên dữ liệu mới, không chỉ giảm training loss.

## 8. Gradient descent và Newton's method

Gradient descent dùng thông tin bậc nhất cục bộ:

$$
\theta_{t+1}=\theta_t-\eta\nabla J(\theta_t).
$$

Với objective lồi trơn, step size đủ nhỏ giúp giảm loss; với giả định mạnh hơn, ta có thể giới hạn tốc độ tiến đến nghiệm. Step size không xét độ cong vẫn có thể làm một bài toán lồi phân kỳ.

Newton's method dùng độ cong để scale bước:

$$
\theta_{t+1}=\theta_t-[\nabla^2J(\theta_t)]^{-1}\nabla J(\theta_t).
$$

Phương pháp này có thể hội tụ nhanh gần nghiệm ổn định, nhưng tính/lưu Hessian và giải hệ tuyến tính rất tốn kém với mô hình lớn. Vì vậy deep learning thường dùng stochastic first-order optimizer và biến thể thay cho Newton update đầy đủ.

## Luyện tập có đáp án

1. $f(x)=3x+2$ có lồi không? **Đáp án:** có, vì là hàm tuyến tính.
2. Vì sao $X^TX$ positive semidefinite chứng minh least-squares lồi? **Đáp án:** đó là Hessian của loss và quadratic form theo mọi hướng v là $\|Xv\|^2\geq0$.
3. $\alpha_i g_i(\theta)=0$ biểu diễn điều gì? **Đáp án:** ràng buộc bất đẳng thức chưa active có multiplier bằng 0; multiplier dương chỉ có thể xảy ra khi ràng buộc active.
4. Objective lồi có bảo đảm gradient descent với step quá lớn hoạt động không? **Đáp án:** không; step vẫn cần phù hợp curvature/smoothness.
