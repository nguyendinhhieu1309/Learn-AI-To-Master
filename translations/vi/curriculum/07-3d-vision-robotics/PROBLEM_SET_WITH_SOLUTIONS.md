# 3D Vision và Robotics: Bộ bài tập có lời giải

[Xem bài gốc bằng tiếng Anh](../../../../curriculum/07-3d-vision-robotics/PROBLEM_SET_WITH_SOLUTIONS.md)

Nếu đề không nói khác, dùng column vector và homogeneous transform. Ghi frame subscript khi giải; nhiều lỗi transform thực chất là lỗi chiều.

## 1. Chiều transform

$T_{A\leftarrow B}$ ánh xạ điểm biểu diễn trong frame B sang tọa độ frame A. Viết công thức.

### Lời giải

$p_A=T_{A\leftarrow B}p_B$. Chỉ số bên phải là source frame; bên trái là destination. Ghi convention này trong code và sơ đồ.

## 2. Ghép transform chain

Cho $T_{base\leftarrow end}$ và $T_{end\leftarrow camera}$, tìm transform từ camera coordinate sang base coordinate.

### Lời giải

$T_{base\leftarrow camera}=T_{base\leftarrow end}T_{end\leftarrow camera}$. Với column vector, transform bên phải tác dụng trước. Kiểm tra bằng điểm đã biết.

## 3. Nghịch đảo rigid transform

Với $T=[R\ t;0\ 1]$, viết inverse.

### Lời giải

Với rigid rotation, inverse của R là $R^\top$. Vậy:

$$
T^{-1}=\begin{bmatrix}R^\top&-R^\top t\\0&1\end{bmatrix}.
$$

## 4. Stereo disparity sang depth

Stereo camera đã rectified có f=400 pixel, baseline B=0.12 m, disparity d=24 pixel. Ước lượng depth Z.

### Lời giải

$Z=fB/d=400\times0.12/24=2$ m. Calibration và disparity uncertainty quan trọng; uncertainty depth tăng mạnh khi disparity nhỏ.

## 5. Voxel downsampling mặt phẳng

Point cloud phủ mặt phẳng 2 m × 2 m. Voxel lập phương cạnh 0.1 m. Xấp xỉ có bao nhiêu voxel được chiếm, bỏ qua biên và khe?

### Lời giải

Diện tích khoảng 4 m². Mỗi voxel cắt mặt phẳng trên diện tích khoảng 0.01 m² nên có khoảng 400 cell được chiếm. Ước lượng thể tích là sai vì point nằm trên mặt phẳng.

## 6. Ước lượng surface normal

Eigenvalue local covariance là 0.02, 0.01 và 0.0001. Eigenvector nào ước lượng normal và spectrum gợi ý gì?

### Lời giải

Eigenvector eigenvalue nhỏ nhất ước lượng normal vì biến thiên nhỏ nhất vuông góc mặt phẳng cục bộ. Spectrum gợi ý neighborhood gần phẳng. Hướng normal cần viewpoint hoặc quy tắc lan truyền để xác định.

## 7. Kabsch reflection correction

SVD alignment cho determinant(R)=-1. Cần correction nào?

### Lời giải

Kết quả thô là reflection. Chèn $D=\operatorname{diag}(1,1,\det(VU^\top))$ rồi tính $R=VDU^\top$ khi $H=U\Sigma V^\top$. Điều này ép determinant(R)=+1. Kiểm tra convention covariance trong code.

## 8. ICP mơ hồ

ICP báo residual thấp cho hai kệ song song lặp lại nhưng pose sai dọc theo hướng kệ. Vì sao?

### Lời giải

Hình học lặp ràng buộc yếu chuyển động theo hướng đối xứng nên nhiều pose có nearest-neighbor distance gần nhau. Residual thấp không chứng minh alignment toàn cục duy nhất. Thêm hình học đặc trưng, pose prior, ngữ cảnh rộng hơn hoặc xác minh độc lập.

## 9. Số vòng RANSAC

Model cần 3 điểm mỗi sample. Tỷ lệ inlier w=0.5 và xác suất mong muốn p=0.99. Dùng $N=\log(1-p)/\log(1-w^s)$.

### Lời giải

Xác suất sample toàn inlier là $0.5^3=0.125$. $N=\log(0.01)/\log(0.875)\approx34.5$, vậy dùng ít nhất 35 vòng theo giả định. Nếu đánh giá w quá cao, xác suất thành công thực tế thấp hơn.

## 10. Giới hạn của Chamfer distance

Hai cloud có symmetric Chamfer distance nhỏ. Điều đó chứng minh correspondence one-to-one hoặc topology giống nhau không?

### Lời giải

Không. Chamfer đo khoảng cách nearest-neighbor trung bình theo cả hai chiều. Nó không chứng minh correspondence một-một, topology, ngữ nghĩa đúng hay không có lỗi cục bộ bị trung bình che mất. Kiểm tra coverage, outlier và geometry theo tác vụ.

## 11. Biến đổi một điểm

Điểm $p_{camera}=[1,0,0]$ m. Translation camera trong base frame là [2,0,0] m, không rotation. Tìm $p_{base}$.

### Lời giải

Dùng $T_{base\leftarrow camera}$ ta được $p_{base}=[3,0,0]$ m. Xác nhận chiều transform trước khi áp translation.

## 12. Split point-cloud scan

Bạn tạo 100 cloud augmentation từ mỗi 20 scan rồi chia ngẫu nhiên tất cả cloud. Vì sao không hợp lệ để test scan chưa thấy?

### Lời giải

Biến thể từ một scan có thể vào cả hai partition, cùng chia sẻ geometry/sensor artifact. Split theo scan gốc, scene hoặc capture session trước; chỉ augmentation training.

## 13. Depth uncertainty

Với $Z=fB/d$, lấy đạo hàm theo disparity d. Độ nhạy đổi ra sao khi d nhỏ?

### Lời giải

$$
\frac{dZ}{dd}=-\frac{fB}{d^2}.
$$

Độ lớn tăng khi disparity giảm nên điểm xa nhạy hơn với cùng sai số disparity. Báo valid range và uncertainty.

## 14. An toàn robot

Point-cloud detector báo xác suất class cao cho vật cản. Đã đủ để điều khiển robot tránh nó chưa?

### Lời giải

Chưa. Class confidence không chứng minh geometric accuracy, free-space safety, calibration, chuyển động vật cản hay collision-free motion. Validation localization/perception, truyền uncertainty, dùng planner được kiểm tra riêng và định nghĩa safe stop.

## Checklist ôn tập

- Tên transform có nêu source và destination không?
- Calibration, synchronization và quy ước coordinate có được ghi không?
- Registration có khởi tạo robust và xử lý outlier không?
- Ambiguity, mật độ, occlusion và uncertainty có được đo không?
- Point-cloud split có group theo scene/scan gốc không?
- Perception có tách khỏi planning/safety đã validation không?
