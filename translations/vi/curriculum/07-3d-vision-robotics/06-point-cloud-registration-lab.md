# Lab point-cloud registration: frame, sampling, normal và ICP

[Xem bài gốc bằng tiếng Anh](../../../../curriculum/07-3d-vision-robotics/06-point-cloud-registration-lab.md)

Bài này nối hình học với workflow point cloud. Registration ước lượng quan hệ giữa hai quan sát từng phần; thách thức không chỉ là giải transform mà còn phải chọn correspondence và coordinate frame hợp lệ.

## Mục tiêu học tập

- Theo dõi source/destination frame của rigid transform.
- Giải thích voxel sampling, neighborhood normal và outlier filtering.
- Suy ra bước rigid alignment dạng đóng dùng trong point-to-point ICP.
- Đánh giá registration bằng metric xét overlap và kiểm tra failure mode.

## Điều kiện tiên quyết

Học [coordinate frame và camera geometry](01-coordinate-frames-and-camera-geometry.md), [RGB-D và point cloud](02-rgbd-point-clouds-and-robot-pipeline.md), [xử lý point cloud](05-point-cloud-processing-and-deep-learning.md).

## 1. Transform luôn có chiều

Gọi $p_B$ là điểm biểu diễn trong frame B. Transform $T_{A\leftarrow B}$ ánh xạ nó sang frame A:

$$
\tilde p_A=T_{A\leftarrow B}\tilde p_B,\qquad
T_{A\leftarrow B}=
\begin{bmatrix}R_{A\leftarrow B}&t_{A\leftarrow B}\\0&1\end{bmatrix}.
$$

Dấu ngã là homogeneous coordinate $[x,y,z,1]^\top$. Rotation cần thỏa $R^\top R=I$ và $\det(R)=+1$.

### Transform ví dụ

Quay 90 độ ngược chiều kim đồng hồ quanh z rồi tịnh tiến $(1,0,0)$:

$$
T_{A\leftarrow B}=
\begin{bmatrix}
0&-1&0&1\\
1&0&0&0\\
0&0&1&0\\
0&0&0&1
\end{bmatrix}.
$$

Với $p_B=(1,0,2)$, quay trước thành $(0,1,2)$, rồi tịnh tiến thành $p_A=(1,1,2)$. Ghi chiều trong tên biến; ký hiệu T trơ rất dễ bị lấy nghịch đảo nhầm.

## 2. Xây preprocessing ổn định

1. Xác minh frame, đơn vị, timestamp và metadata sensor.
2. Bỏ point không hợp lệ và artifact đặc thù sensor theo quy tắc đã ghi.
3. Voxel-downsample để giảm mật độ mà vẫn giữ gần đúng hình học.
4. Ước lượng normal từ neighborhood nếu phương pháp sau cần.
5. Tìm correspondence ứng viên bằng descriptor hình học hoặc learned descriptor.
6. Loại outlier trước khi giải rigid transform.
7. Tinh chỉnh transform và đánh giá trên điểm held-out hoặc lấy mẫu độc lập.

Voxel size dùng cùng đơn vị với cloud. Giá trị 0.05 chỉ là 5 cm khi tọa độ dùng mét.

### Normal của neighborhood

Với một điểm và neighborhood, center các neighbor rồi tính covariance:

$$
C=\frac1k\sum_{j=1}^{k}(q_j-\bar q)(q_j-\bar q)^\top.
$$

Eigenvector ứng với eigenvalue nhỏ nhất ước lượng local surface normal. Cách này giả định neighborhood đủ phẳng và không bị noise/hỗn hợp bề mặt chi phối.

Neighborhood quá nhỏ làm normal bất ổn; quá lớn làm mượt mất góc và cấu trúc mảnh. Kiểm tra neighborhood ở nhiều vị trí và scale.

## 3. Point-to-point rigid alignment

Với source point tương ứng $p_i$ và destination point $q_i$, giải:

$$
\min_{R,t}\sum_i\|Rp_i+t-q_i\|_2^2
\quad\text{subject to}\quad
R^\top R=I,\ \det(R)=1.
$$

Tính centroid $\bar p,\bar q$, tọa độ đã center $x_i=p_i-\bar p$, $y_i=q_i-\bar q$. Tạo:

$$
H=\sum_i x_i y_i^\top,\qquad H=U\Sigma V^\top.
$$

Khi đó:

$$
R=V\operatorname{diag}(1,1,\det(VU^\top))U^\top,\qquad
t=\bar q-R\bar p.
$$

Determinant correction ngăn reflection. Công thức này giải transform least-squares cho correspondence đã cho; nó không tự tìm correspondence đúng.

## 4. Iterative Closest Point (ICP)

ICP lặp:

1. transform source point bằng estimate hiện tại;
2. tìm target point gần nhất;
3. loại match không hợp lệ hoặc quá xa;
4. giải rigid alignment cho cặp còn lại;
5. ghép update vào transform hiện tại;
6. dừng khi update/objective đổi ít hoặc đạt giới hạn vòng lặp.

Point-to-plane ICP tối thiểu hóa:

$$
\sum_i\left(n_i^\top(Rp_i+t-q_i)\right)^2
$$

với $n_i$ là target surface normal. Nó có thể hội tụ nhanh khi pose ban đầu tốt trên bề mặt trơn, nhưng nhạy hơn với normal sai và khởi tạo kém.

ICP là phương pháp cục bộ: khởi tạo tệ, overlap ít, đối xứng, hình học lặp, object động và trộn đơn vị đều có thể tạo alignment trông hợp lý nhưng sai. Residual nhỏ trên overlap rất ít không chứng minh đúng toàn cục.

## 5. Metric và điều chúng không thể nói

Với hai point set P và Q, Chamfer-style distance đối xứng:

$$
d_{\mathrm{ch}}(P,Q)=
\frac1{|P|}\sum_{p\in P}\min_{q\in Q}\|p-q\|_2^2+
\frac1{|Q|}\sum_{q\in Q}\min_{p\in P}\|q-p\|_2^2.
$$

Metric này phụ thuộc mật độ sampling và có thể che lỗi cục bộ trong cloud lớn. Cũng báo overlap hoặc inlier ratio, transform error khi có ground truth và failure rate giữa scene. Dùng threshold theo đơn vị vật lý có tài liệu.

Với point-cloud learning, báo thêm task metric: per-class IoU cho segmentation, average precision cho detection và split theo scene/sequence. Split random theo point có thể leak bề mặt gần như giống nhau.

## Bài luyện tập kèm đáp án

1. LiDAR cloud dùng millimeter nhưng voxel size bằng 0.05. Điều gì có thể xảy ra?
2. Vì sao cần determinant correction trong SVD alignment?
3. Hai cloud chỉ overlap 10%. Vì sao nearest-neighbor ICP có thể thất bại dù update toán học đúng?
4. Point-to-plane ICP khác point-to-point ICP thế nào?
5. Vì sao Chamfer distance thấp chưa đủ tuyên bố robot pose estimate an toàn?

### Đáp án

1. Voxel size quá nhỏ so với tọa độ millimeter, có thể gần như không downsample. Đổi đơn vị hoặc chọn scale thống nhất.
2. Nghiệm SVD chưa ràng buộc có thể chứa reflection determinant -1, không phải rotation hợp lệ.
3. Phần lớn nearest-neighbor pair có thể sai. Optimizer tối thiểu hóa lỗi của các cặp sai rồi hội tụ vào alignment cục bộ khác.
4. Point-to-point tối thiểu hóa độ dời Euclidean; point-to-plane tối thiểu hóa độ dời chiếu theo target normal.
5. Nó có thể che overlap thấp, đối xứng, lỗi cục bộ, transform sai chiều hoặc failure nguy hiểm ở vùng robot cần quan tâm.

## Mini-project

Dùng hai partial scan của cùng scene. Ghi coordinate frame/đơn vị; trực quan hóa trước-sau; so ít nhất hai cách khởi tạo/correspondence; báo overlap, residual, tính hợp lý của transform và trường hợp lỗi. Đính kèm trình tự preprocessing và lựa chọn parameter có thể lặp lại.

## Lỗi thường gặp

- Áp transform ngược chiều.
- Trộn mét với millimeter.
- Tính lại nearest neighbor/correspondence mà không loại outlier.
- Coi ICP hội tụ là bằng chứng pose đúng toàn cục.
- Chỉ báo distance trung bình mà thiếu overlap hoặc failure rate.
