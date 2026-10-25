# Cheat sheet công thức Toán cho AI

> Bản tiếng Anh: [AI Mathematics Formula Cheatsheet](../../../../curriculum/01-math/FORMULA_CHEATSHEET.md)

Dùng trang này để tra nhanh sau khi đã học bài tương ứng. Một ký hiệu có thể mang nghĩa khác nhau giữa các lĩnh vực, nên luôn kiểm tra định nghĩa tại chỗ.

## Đại số tuyến tính

Tích vô hướng và cosine similarity:

$$
x^\top y = \sum_{i=1}^{d}x_i y_i,
\qquad
\cos(x,y)=\frac{x^\top y}{\|x\|_2\,\|y\|_2}
$$

Dot product kết hợp các tọa độ tương ứng; cosine similarity so sánh hướng và bỏ qua độ lớn.

Shape phép nhân ma trận:

$$
A\in\mathbb{R}^{m\times n},\quad
x\in\mathbb{R}^{n}
\quad\Longrightarrow\quad
Ax\in\mathbb{R}^{m}
$$

Các kích thước trong phải khớp. Với batch gồm vector hàng $X\in\mathbb{R}^{b\times n}$ và weight $W\in\mathbb{R}^{n\times k}$, output là $XW\in\mathbb{R}^{b\times k}$.

## Giải tích và tối ưu hóa

Gradient descent:

$$
\theta_{t+1}=\theta_t-\eta\nabla_\theta J(\theta_t)
$$

Gradient chỉ hướng tăng dốc nhất cục bộ; trừ gradient để giảm objective. Learning rate $\eta$ kiểm soát độ dài bước.

Chain rule:

$$
\frac{\partial L}{\partial x}
=\frac{\partial L}{\partial y}\frac{\partial y}{\partial x}
$$

Backpropagation áp dụng chain rule lặp lại qua computational graph.

Activation và đạo hàm thường gặp:

$$
\sigma(z)=\frac{1}{1+e^{-z}},
\qquad \sigma'(z)=\sigma(z)(1-\sigma(z)),
\qquad \operatorname{ReLU}(z)=\max(0,z)
$$

Với ReLU, đạo hàm là 1 khi $z>0$ và 0 khi $z<0$; tại $z=0$, implementation chọn một quy ước.

## Xác suất và thống kê

Bayes' rule:

$$
P(A\mid B)=\frac{P(B\mid A)P(A)}{P(B)}
$$

Cập nhật niềm tin ban đầu bằng likelihood của bằng chứng quan sát.

Expectation, variance và standard error:

$$
\mathbb{E}[X]=\sum_x xP(X=x),\qquad
\operatorname{Var}(X)=\mathbb{E}[(X-\mathbb{E}[X])^2],\qquad
SE(\bar{x})=\frac{s}{\sqrt{n}}
$$

Variance đo độ phân tán của từng giá trị; standard error đo độ bất định của sample mean dưới giả định lấy mẫu.

Bernoulli probability và log-likelihood cho quan sát nhị phân:

$$
P(X=x)=p^x(1-p)^{1-x},
\qquad
\ell(p)=\sum_{i=1}^{N}\left[x_i\log p+(1-x_i)\log(1-p)\right]
$$

Với các lần tung độc lập, MLE là $\hat p=\frac{1}{N}\sum_i x_i$. Logarithm đổi tích xác suất thành tổng.

Maximum likelihood và MAP:

$$
\hat\theta_{\mathrm{MLE}}=\arg\min_\theta-\log p(D\mid\theta),
\qquad
\hat\theta_{\mathrm{MAP}}=\arg\min_\theta\left[-\log p(D\mid\theta)-\log p(\theta)\right]
$$

MAP thêm penalty từ prior; Gaussian prior quanh 0 trên weight tương ứng penalty $L_2$.

## Supervised learning

Mean squared error:

$$
\operatorname{MSE}=\frac{1}{N}\sum_{i=1}^{N}(y_i-\hat{y}_i)^2
$$

Binary cross-entropy với xác suất dự đoán $p_i$ và binary label $y_i$:

$$
L=-\frac{1}{N}\sum_{i=1}^{N}
\left[y_i\log p_i+(1-y_i)\log(1-p_i)\right]
$$

Empirical risk có regularization:

$$
J(\theta)=\frac{1}{N}\sum_{i=1}^{N}\ell(f_\theta(x_i),y_i)
+\lambda\Omega(\theta)
$$

Thành phần đầu fit ví dụ quan sát; thành phần sau phạt độ phức tạp theo penalty đã chọn.

Linear regression với $X\in\mathbb{R}^{N\times d}$ và residual $r=Xw+b\mathbf{1}-y$:

$$
J(w,b)=\frac{1}{2N}\|r\|_2^2,
\qquad
\nabla_wJ=\frac{1}{N}X^Tr,
\qquad
\frac{\partial J}{\partial b}=\frac{1}{N}\mathbf{1}^Tr
$$

Logistic regression với $p=\sigma(Xw+b\mathbf{1})$:

$$
\nabla_wJ=\frac{1}{N}X^T(p-y),
\qquad
\frac{\partial J}{\partial b}=\frac{1}{N}\mathbf{1}^T(p-y)
$$

Objective k-means:

$$
J=\sum_{i=1}^{N}\|x_i-\mu_{c_i}\|_2^2
$$

Gán mỗi điểm vào centroid gần nhất, sau đó thay centroid bằng mean của cluster.

SVM hinge loss với $y_i\in\{-1,+1\}$ và score $s_i=w^Tx_i+b$:

$$
\ell_i=\max(0,1-y_is_i)
$$

Penalty margin bằng 0 khi ví dụ được phân loại đúng và cách ranh giới ít nhất một normalized margin.

Phần explained variance được giữ bởi k principal component đầu:

$$
\operatorname{EVR}(k)=\frac{\sum_{j=1}^{k}\lambda_j}{\sum_{j=1}^{d}\lambda_j},
\qquad \lambda_j=\frac{\sigma_j^2}{N-1}
$$

Ở đây $\lambda_j$ là eigenvalue của covariance và $\sigma_j$ là singular value của dữ liệu đã center.

## Metric classification

$$
\operatorname{Precision}=\frac{TP}{TP+FP},\qquad
\operatorname{Recall}=\frac{TP}{TP+FN},\qquad
F_1=2\frac{\operatorname{Precision}\cdot\operatorname{Recall}}
{\operatorname{Precision}+\operatorname{Recall}}
$$

Precision hỏi “trong các dự đoán positive, có bao nhiêu đúng?”; recall hỏi “trong các positive thật, tìm được bao nhiêu?”. Luôn định nghĩa positive class và báo cáo số đếm khi dữ liệu mất cân bằng.

## Neural network và attention

Softmax chuyển logits $z$ thành categorical distribution:

$$
\operatorname{softmax}(z_i)=\frac{e^{z_i}}{\sum_j e^{z_j}}
$$

Với target one-hot y, softmax kết hợp cross-entropy có gradient theo logit đơn giản:

$$
\frac{\partial L}{\partial z_i}=p_i-y_i
$$

trong đó $p_i=\operatorname{softmax}(z)_i$. Hàm cross-entropy của framework thường nhận raw logits và tính tổ hợp này ổn định.

Scaled dot-product attention:

$$
\operatorname{Attention}(Q,K,V)
=\operatorname{softmax}\left(\frac{QK^\top}{\sqrt{d_k}}\right)V
$$

Query chọn key; trọng số attention thu được dùng để trộn value vector. Scale $\sqrt{d_k}$ giúp dot product nằm trong miền số học hữu ích.

Kích thước output convolution theo chiều dài đầu vào H, kernel K, padding P, stride S và dilation D:

$$
H_{\mathrm{out}}=\left\lfloor\frac{H+2P-D(K-1)-1}{S}\right\rfloor+1
$$

Áp dụng cùng công thức cho width. Với convolution chuẩn có $C_{in}$ channel đầu vào, $C_{out}$ filter và kernel $K_h\times K_w$, số tham số gồm bias là $C_{out}(C_{in}K_hK_w+1)$.

## Computer Vision và 3D

Intersection over Union:

$$
\operatorname{IoU}(A,B)=\frac{|A\cap B|}{|A\cup B|}
$$

Pinhole camera projection lý tưởng:

$$
u=f_x\frac{X}{Z}+c_x,\qquad
v=f_y\frac{Y}{Z}+c_y
$$

Depth từ stereo camera đã rectified:

$$
Z=\frac{fB}{d}
$$

Trong đó f là focal length theo pixel, B là baseline camera, d là disparity theo pixel. Đơn vị phải nhất quán.

## Reinforcement Learning

Discounted return:

$$
G_t=\sum_{k=0}^{\infty}\gamma^k r_{t+k}
$$

Bellman optimality equation:

$$
Q^*(s,a)=\mathbb{E}\left[r+\gamma\max_{a'}Q^*(s',a')\mid s,a\right]
$$

Giá trị hành động gồm reward ngay lập tức và giá trị đã chiết khấu của state tiếp theo.

## Ký hiệu và lỗi thường gặp

- N thường là số ví dụ; d là số chiều feature; b là batch size; $\theta$ là tham số mô hình.
- Phân biệt probability p với logit z; hàm cross-entropy thường nhận logits.
- Nêu metric được lấy trung bình theo mẫu, lớp, nhóm hay threshold.
- Ghi đơn vị cho đại lượng vật lý và coordinate frame cho điểm 3D.
- Công thức gồm cả giả định; tính độc lập, distribution và cách lấy mẫu đều quan trọng.
