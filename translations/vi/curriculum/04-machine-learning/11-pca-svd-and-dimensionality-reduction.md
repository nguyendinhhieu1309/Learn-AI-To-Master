# PCA, SVD và dimensionality reduction

[Xem bài gốc bằng tiếng Anh](../../../../curriculum/04-machine-learning/11-pca-svd-and-dimensionality-reduction.md)

Principal Component Analysis (PCA) tìm các hướng trực giao giữ được nhiều variance nhất dưới phép chiếu tuyến tính. PCA hữu ích cho nén, trực quan hóa, khử nhiễu và preprocessing — nhưng không khám phá label hay bảo đảm model tốt hơn.

## Mục tiêu học tập

- Suy ra principal component đầu tiên dưới dạng hướng có variance tối đa.
- Liên hệ eigenvector của covariance với Singular Value Decomposition.
- Tính explained variance và chiếu/tái dựng một điểm.
- Fit và dùng PCA mà không leak thông tin validation/test.
- Giải thích giới hạn và so sánh PCA với lựa chọn supervised.

## 1. Center training data

Cho $X\in\mathbb{R}^{n\times d}$ chứa n quan sát theo hàng, d feature theo cột. Tính mean cho từng feature từ training example rồi center:

$$
X_c=X-\mathbf{1}\mu^T,
\qquad
\mu_j=\frac1n\sum_{i=1}^{n}X_{ij}.
$$

PCA dựa trên variance quanh mean. Nếu không trừ feature mean, component đầu tiên có thể chủ yếu hướng về offset của data. Covariance matrix là:

$$
S=\frac{1}{n-1}X_c^TX_c\in\mathbb{R}^{d\times d}.
$$

## 2. Suy ra component đầu tiên

Chiếu row vector đã center $x_i$ lên hướng đơn vị $u$. Tọa độ scalar là $x_i^Tu$. Tổng sample variance của các tọa độ chiếu:

$$
\frac{1}{n-1}\sum_i(x_i^Tu)^2=u^TSu.
$$

Principal direction đầu tiên giải:

$$
u_1=\arg\max_{\|u\|_2=1}u^TSu.
$$

Dùng Lagrange multiplier suy ra $Su_1=\lambda_1u_1$: hướng đó là eigenvector của covariance matrix ứng với eigenvalue lớn nhất. Component sau tối đa hóa variance còn lại và giữ trực giao với component trước.

## 3. Ví dụ hai feature có hướng dẫn

Lấy bốn điểm đã center:

$$
X_c=\begin{bmatrix}1&0\\-1&0\\0&2\\0&-2\end{bmatrix}.
$$

Sample covariance và eigenvalue:

$$
S=\frac13X_c^TX_c=
\begin{bmatrix}2/3&0\\0&8/3\end{bmatrix},
\qquad \lambda_1=8/3,\;\lambda_2=2/3.
$$

Component đầu tiên hướng theo feature thứ hai. Nó giữ $8/10=80\%$ tổng variance. Điểm $(3,4)^T$ chiếu thành tọa độ 4 theo $u_1=(0,1)^T$ và tái dựng thành $(0,4)^T$. Thành phần ngang bị mất, vì vậy phép chiếu làm mất thông tin.

## 4. Explained variance và chọn số component

Nếu eigenvalue sắp giảm dần, variance giữ lại bởi k component đầu là:

$$
\operatorname{explained\ variance\ ratio}(k)
=\frac{\sum_{j=1}^{k}\lambda_j}{\sum_{j=1}^{d}\lambda_j}.
$$

Vẽ cumulative explained variance và validation performance của tác vụ downstream. Ngưỡng như 95% là heuristic nén, không phải quy tắc phổ quát giữ thông tin dự đoán: hướng variance thấp vẫn có thể chứa tín hiệu label hữu ích.

## 5. SVD là hướng tính toán ổn định

Với dữ liệu đã center, tính thin singular value decomposition:

$$
X_c=U\Sigma V^T.
$$

Cột của $V$ là principal direction trong feature space. Singular value $\sigma_j$ liên hệ với eigenvalue covariance:

$$
\lambda_j=\frac{\sigma_j^2}{n-1}.
$$

Giữ k cột đầu $V_k$ để biến đổi row-vector example thành $z=x_c^TV_k$ (tương đương batch theo hàng: $Z=X_cV_k$). Tái dựng là $\hat X_c=ZV_k^T$. SVD dùng được với matrix chữ nhật và thường ổn định số hơn việc tạo $X_c^TX_c$ trước.

## 6. Scale hay không scale?

PCA trên đơn vị thô để feature có variance lớn ảnh hưởng nhiều hơn. Standardize từng feature về variance đơn vị đổi câu hỏi thành phân tích tương quan thay vì covariance thô. Dùng standardization khi chênh lệch đơn vị tùy ý và mong muốn scale so sánh được. Đừng standardize máy móc nếu độ lớn tuyệt đối mang tín hiệu có ý nghĩa. Quyết định dựa trên định nghĩa feature rồi validation toàn pipeline.

## 7. Tránh leakage

Feature mean, thống kê scaling và principal component đều được học từ dữ liệu. Fit scaler và PCA **chỉ trên từng training fold**, rồi dùng các object đã fit transform validation example. Trong scikit-learn, đặt StandardScaler và PCA cùng estimator trong Pipeline để cross-validation fit lại đúng cách.

~~~python
from sklearn.decomposition import PCA
from sklearn.pipeline import make_pipeline
from sklearn.preprocessing import StandardScaler
from sklearn.linear_model import LogisticRegression

model = make_pipeline(
    StandardScaler(),
    PCA(n_components=0.95),
    LogisticRegression(max_iter=1000),
)
model.fit(X_train, y_train)
~~~

Setting 0.95 giữ đủ component giải thích 95% variance của training data. Nó không bảo đảm classification accuracy 95%.

## 8. Hạn chế và lựa chọn khác

- PCA là tuyến tính, giữ hướng variance cao chứ không nhất thiết giữ class separation hay cấu trúc liên quan tác vụ.
- Component khó diễn giải vì mỗi component kết hợp feature gốc.
- Outlier có thể ảnh hưởng mạnh mean và covariance.
- PCA có thể giảm noise/compute nhưng cũng loại bỏ tín hiệu hữu ích.
- LDA là supervised, tìm hướng phân biệt class có nhãn; nó trả lời câu hỏi khác.
- Random projection, autoencoder và nonlinear manifold method dựa trên giả định khác nhau, cần đánh giá theo tác vụ.

## Bài luyện tập kèm đáp án

1. Eigenvalue $\lambda_j$ biểu diễn gì trong covariance PCA? **Đáp án:** variance theo eigenvector đơn vị tương ứng.
2. Nếu eigenvalue là 9 và 1, PC1 giữ bao nhiêu variance? **Đáp án:** $9/(9+1)=90\%$.
3. Giữ 95% variance feature có bảo đảm giữ 95% predictive accuracy không? **Đáp án:** không.
4. Vì sao fit PCA bên trong từng training fold? **Đáp án:** validation/test không được ảnh hưởng hướng PCA hay scaling đã học.
