# k-nearest neighbors và k-means

[Xem bài gốc bằng tiếng Anh](../../../../curriculum/04-machine-learning/09-knn-and-k-means.md)

Tên hai phương pháp giống nhau nhưng tác vụ khác nhau. **k-nearest neighbors (k-NN)** dự đoán target dựa trên ví dụ có nhãn ở gần. **k-means** gom ví dụ chưa có nhãn quanh centroid được học.

## Mục tiêu học tập

- Giải thích training và prediction của k-NN, k-means.
- Tự tính ví dụ dự đoán nhỏ dựa trên distance.
- Nêu vì sao feature scaling và lựa chọn distance quan trọng.
- Suy ra objective k-means và các bước cập nhật luân phiên.
- Nhận ra khi hình học dữ liệu không phù hợp với các phương pháp này.

## 1. Distance là một lựa chọn modeling

Với vector $x,z\in\mathbb{R}^d$, Euclidean distance là:

$$
d_2(x,z)=\sqrt{\sum_{j=1}^{d}(x_j-z_j)^2}.
$$

Manhattan distance cộng độ lớn chênh lệch từng tọa độ; cosine distance tập trung vào góc. Distance định nghĩa trường hợp nào được xem là “tương tự”. Nếu tuổi nằm trong 0–100 còn thu nhập hằng năm trong 0–1,000,000, Euclidean distance có thể gần như chỉ phản ánh thu nhập. Fit scaling trên training data rồi áp dụng không đổi khi evaluation và inference.

## 2. k-nearest neighbors (k-NN)

Để dự đoán ví dụ mới:

1. tính distance từ nó tới từng training example;
2. chọn k ví dụ có nhãn gần nhất;
3. với classification, lấy majority vote hoặc vote có trọng số distance;
4. với regression, lấy trung bình target của các láng giềng, có thể dùng distance weight.

### Ví dụ classification có hướng dẫn

Giả sử training point một chiều là $0\mapsto A$, $3\mapsto B$ và $4\mapsto B$. Với query $x=1.2$, distance là 1.2, 1.8 và 2.8. Khi $k=1$, dự đoán A; khi $k=3$, majority là B. Tăng k làm quyết định cục bộ trơn hơn nhưng có thể xóa pattern nhỏ. Chọn k bằng validation data.

k nhỏ có bias thấp và variance cao; k lớn thường bias cao hơn, variance thấp hơn. Trong không gian nhiều chiều, “gần nhất” có thể không còn nghĩa và prediction tốn kém vì phải tính nhiều distance. Approximate-nearest-neighbor index tăng tốc retrieval quy mô lớn nhưng có thể đánh đổi độ chính xác để giảm latency.

## 3. Phân cụm k-means

Với n điểm chưa gán nhãn $x_i$ và số cluster K đã chọn, k-means học centroid $\mu_k$ và assignment $c_i$. Thuật toán tối thiểu hóa tổng bình phương Euclidean distance trong cluster:

$$
J(c,\mu)=\sum_{i=1}^{n}\|x_i-\mu_{c_i}\|_2^2.
$$

Thuật toán luân phiên hai bước:

1. **Assign:** gán mỗi điểm cho centroid gần nhất.
2. **Update:** thay centroid bằng mean của các điểm được gán.

Mỗi bước không thể làm objective tăng khi giữ nguyên biến còn lại; vì vậy objective giảm hoặc giữ nguyên cho đến khi assignment không đổi. Objective toàn cục không lồi; centroid khởi đầu khác nhau có thể đưa đến nghiệm cục bộ khác nhau.

### Ví dụ cập nhật trên bốn điểm

Dùng điểm một chiều $0,2,8,10$ và $K=2$. Khởi tạo centroid tại 0 và 10. Assignment là $\{0,2\}$ và $\{8,10\}$. Mean mới là 1 và 9. Gán lại vẫn giữ nguyên hai nhóm. Tổng bình phương distance là:

$$
(0-1)^2+(2-1)^2+(8-9)^2+(10-9)^2=4.
$$

Giá trị này mô tả độ gọn theo feature representation đã chọn; nó không chứng minh dữ liệu thực sự có hai nhóm tự nhiên.

## 4. Chọn k và đánh giá clustering cẩn thận

Với k-NN, tune k, distance, weight và preprocessing trong train/validation split. Với k-means, inertia luôn giảm khi K tăng, nên chỉ nhìn inertia thấp nhất không giúp chọn số cluster hữu ích. So sánh elbow plot, silhouette score, độ ổn định khi resampling, ích lợi downstream và khả năng chuyên gia domain hiểu cluster. Nếu có label, so với external metric nhưng đừng coi objective unsupervised như classification accuracy.

## 5. Hạn chế và lựa chọn khác

- k-NN có thể dự đoán chậm và nhạy với chiều không liên quan, scale, ví dụ trùng.
- k-means ưu tiên cluster gần hình cầu, scale tương đương theo Euclidean distance. Nó khó xử lý hình thuôn dài, mật độ rất khác nhau và nhiều outlier.
- Centroid là mean nên outlier có thể kéo lệch; k-medoids hoặc phương pháp robust có thể tốt hơn.
- Density-based clustering tìm được hình dạng bất quy tắc và đánh dấu noise; hierarchical clustering cho thấy cấu trúc đa mức.

## 6. Mẫu Python

~~~python
from sklearn.pipeline import make_pipeline
from sklearn.preprocessing import StandardScaler
from sklearn.neighbors import KNeighborsClassifier

knn = make_pipeline(StandardScaler(), KNeighborsClassifier(n_neighbors=5))
knn.fit(X_train, y_train)
~~~

~~~python
from sklearn.cluster import KMeans
from sklearn.preprocessing import StandardScaler

scaler = StandardScaler().fit(X_train)
X_scaled = scaler.transform(X_train)
clusters = KMeans(n_clusters=3, n_init="auto", random_state=0).fit(X_scaled)
~~~

Không fit scaler trên dữ liệu evaluation tương lai. Dùng nhiều lần khởi tạo k-means vì một lần chạy có thể dừng ở nghiệm cục bộ kém.

## Bài luyện tập kèm đáp án

1. Một feature đo bằng đô-la, feature khác bằng năm. Cần kiểm tra gì trước khi học theo Euclidean distance? **Đáp án:** scaling, mức liên quan của feature và distance có phản ánh độ tương tự có ý nghĩa không.
2. Centroid mới của cluster $\{1,3,8\}$ và $\{10,12\}$ là gì? **Đáp án:** 4 và 11.
3. Inertia chứng minh cluster là nhóm có thật ngoài đời không? **Đáp án:** không; nó chỉ đo objective hình học đã chọn.
4. k lớn hơn thường làm gì trong k-NN? **Đáp án:** làm prediction trơn hơn, thường giảm variance nhưng tăng bias.
