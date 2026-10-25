# Đại số tuyến tính cho Machine Learning

> Bản tiếng Anh: [Linear Algebra for Machine Learning](../../../../curriculum/01-math/05-linear-algebra-for-machine-learning.md)

Bài học này liên hệ vector và ma trận với tham số mô hình, embedding, phép chiếu và các vấn đề tính toán số.

## 1. Vector, norm và khoảng cách

Vector là danh sách feature có thứ tự. Norm $L_1$ và $L_2$ đo độ lớn:

$$
\|x\|_1=\sum_i |x_i|,\qquad
\|x\|_2=\sqrt{\sum_i x_i^2}
$$

Penalty $L_1$ khuyến khích hệ số thưa; penalty $L_2$ làm hệ số co nhỏ dần. Khoảng cách Euclid là $\|x-y\|_2$. Scale feature rất quan trọng: nếu một tọa độ tính bằng nghìn còn tọa độ khác là phần lẻ nhỏ, feature có đơn vị lớn có thể chi phối khoảng cách và regularization.

## 2. Ma trận là phép biến đổi

Ma trận ánh xạ vector đầu vào sang một không gian mới. Trong linear layer, $y=Wx+b$; mỗi hàng của W xác định một unit đầu ra. Neural network áp dụng nonlinear activation sau phép biến đổi này. Theo dõi kích thước: nếu $x\in\mathbb{R}^{d}$ và có h hidden unit thì $W\in\mathbb{R}^{h\times d}$ và $Wx\in\mathbb{R}^{h}$.

Với batch, hãy chọn quy ước và giữ nhất quán. Nếu mỗi ví dụ là một hàng, $X\in\mathbb{R}^{b\times d}$ và $W\in\mathbb{R}^{d\times h}$, nên $XW\in\mathbb{R}^{b\times h}$. Lỗi shape thường cho thấy batch/channel axis bị đổi, broadcasting sai hoặc weight matrix bị chuyển vị.

## 3. Rank, cơ sở và phép chiếu

Rank của ma trận là số chiều của không gian do các cột sinh ra. Feature phụ thuộc tuyến tính không thêm hướng tuyến tính mới. Basis là một tập vector độc lập có thể biểu diễn mọi vector trong không gian.

Với vector đơn vị $u$, phép chiếu vô hướng của $x$ lên $u$ là $u^Tx$, vector sau chiếu là $(u^Tx)u$. Chiếu lên không gian con có orthonormal basis dạng ma trận U:

$$
\operatorname{proj}_{U}(x)=UU^Tx
$$

Principal Component Analysis tìm các hướng có phương sai cao rồi chiếu vào không gian ít chiều hơn. Chuẩn hóa feature trước nếu đơn vị hoặc scale khác nhau; đừng chuẩn hóa máy móc nếu scale tuyệt đối mang ý nghĩa.

## 4. Eigenvalue và Singular Value Decomposition

Eigenvector $v$ của ma trận vuông A giữ nguyên hướng sau phép biến đổi: $Av=\lambda v$. Eigenvalue mô tả mức kéo giãn hoặc co lại theo các eigen-direction. Với covariance matrix, eigenvector là hướng chính còn eigenvalue là phương sai theo hướng đó.

Singular Value Decomposition dùng được với mọi ma trận thực:

$$
A=U\Sigma V^T
$$

Các singular value trên đường chéo của $\Sigma$ không âm và theo quy ước được sắp giảm dần. Cắt bỏ singular value nhỏ tạo low-rank approximation hữu ích cho nén và khử nhiễu. Condition number lớn (tỷ số singular value khác 0 lớn nhất với nhỏ nhất) khiến bài toán nhạy với sai số số học.

## 5. Tensor và broadcasting

Tensor là mảng có các trục theo vị trí hoặc tên. Hãy ghi ý nghĩa như batch, height, width, channel, time và feature. Broadcasting căn chỉnh kích thước từ bên phải theo quy tắc của thư viện mảng; code có thể ngắn gọn nhưng thiếu singleton axis đôi khi tạo shape sai mà khó nhận ra.

Ví dụ: trừ mean theo từng feature có shape (features,) khỏi batch matrix (batch, features) là chủ đích. Trừ mean theo từng ví dụ cần shape (batch,1). Kiểm tra shape ở ranh giới dữ liệu và tự tính một ví dụ nhỏ.

## Ví dụ có lời giải: covariance, principal component và phép chiếu

Xét bốn quan sát hai chiều đã được center:

$$
X=\begin{bmatrix}
1&0\\-1&0\\0&2\\0&-2
\end{bmatrix}.
$$

Dùng sample covariance với mẫu số $n-1=3$:

$$
S=\frac{1}{3}X^TX
=\begin{bmatrix}2/3&0\\0&8/3\end{bmatrix}.
$$

Các trục tọa độ là eigenvector. Eigenvalue $2/3$ và $8/3$ là sample variance theo từng hướng. Principal component đầu tiên là trục dọc vì eigenvalue lớn hơn. Chỉ giữ hướng này sẽ bảo toàn:

$$
\frac{8/3}{8/3+2/3}=0.8
$$

tổng phương sai. Với điểm mới $x=(3,4)^T$ và component $u=(0,1)^T$, tọa độ theo component thứ nhất là $u^Tx=4$, còn tái tạo một chiều là $uu^Tx=(0,4)^T$. Thành phần ngang bị bỏ đi chính là thông tin phép chiếu loại bỏ.

Với dữ liệu thật, trước hết trừ mean của tập train. Scale từng feature về unit variance sẽ thay đổi hình học và hướng principal component, vì vậy chỉ standardize khi phù hợp với ý nghĩa feature.

## Least squares là một phép chiếu

Với hệ tuyến tính $Xw\approx y$, đẳng thức chính xác có thể không tồn tại khi quan sát nhiễu hoặc số quan sát nhiều hơn số tham số. Least squares chọn w sao cho dự đoán Xw gần y nhất theo khoảng cách Euclid:

$$
\hat w=\arg\min_w\|Xw-y\|_2^2.
$$

Tại nghiệm tối ưu, residual $r=y-X\hat w$ vuông góc với mọi cột của X, tạo thành normal equation:

$$
X^TX\hat w=X^Ty.
$$

Nếu $X^TX$ khả nghịch, $\hat w=(X^TX)^{-1}X^Ty$. Khi các cột phụ thuộc hoặc số feature nhiều hơn số ví dụ, inverse có thể không tồn tại; hãy dùng least-squares solver hoặc pseudoinverse thay vì tính inverse tường minh. Ridge regression thêm penalty để ổn định các hướng khó xác định:

$$
\hat w_{\mathrm{ridge}}=\arg\min_w\left(\|Xw-y\|_2^2+\lambda\|w\|_2^2\right).
$$

## SVD và PCA

Với data matrix đã center $X\in\mathbb{R}^{n\times d}$, thin SVD là:

$$
X=U\Sigma V^T.
$$

Cột của V là các hướng trực chuẩn trong feature space; singular value trong $\Sigma$ mô tả độ biến thiên của dữ liệu đã center theo từng hướng. Sample covariance:

$$
S=\frac{1}{n-1}X^TX
=V\frac{\Sigma^T\Sigma}{n-1}V^T.
$$

Do đó PCA có thể tính từ eigenvector của covariance hoặc right singular vector của dữ liệu đã center. SVD trực tiếp thường ổn định số tốt hơn tạo $X^TX$, vốn bình phương condition number. Giữ k right singular vector đầu tiên để chiếu dữ liệu vào không gian con k chiều; tái tạo từ chúng là low-rank approximation tốt nhất theo squared error.

## Bài tập

1. X có shape (64,128), linear layer có 32 output. Weight matrix và output có shape nào nếu mỗi hàng là một ví dụ?
2. Vì sao cần scale feature trước k-nearest neighbors hoặc linear model regularization $L_2$?
3. Singular value nhỏ gợi ý điều gì về một hướng trong dữ liệu?
4. Nếu projection matrix được tạo từ các cột không trực chuẩn, vì sao $UU^T$ nói chung không phải projector đúng?
5. Trong ví dụ PCA, giữ component đầu tiên làm mất bao nhiêu phương sai? **Đáp án:** $1-0.8=0.2$, tức 20%.
6. Nếu rank(X)=3, có bao nhiêu singular value khác 0 trong số học chính xác? **Đáp án:** 3.
