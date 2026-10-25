# Stereo, visual odometry, SLAM và point cloud

[Xem bài gốc bằng tiếng Anh](../../../../curriculum/07-3d-vision-robotics/04-stereo-visual-odometry-slam-and-point-clouds.md)

Robot cần nhiều hơn nhận dạng ảnh. Chúng ước lượng hình học, chuyển động và độ bất định khi cảm biến di chuyển cùng nền tảng. Bài này kết nối camera calibration, stereo depth, point-cloud alignment, visual odometry và SLAM.

## 1. Camera calibration và tọa độ ảnh

Intrinsic matrix biểu diễn focal length và principal point:

$$
K = \begin{bmatrix}
f_x & 0 & c_x \\
0 & f_y & c_y \\
0 & 0 & 1
\end{bmatrix}
$$

Lens distortion làm cong pinhole projection lý tưởng, nhất là gần mép ảnh. Ước lượng intrinsic và distortion bằng các ảnh calibration ở nhiều vị trí/hướng của bảng. Reprojection error hữu ích nhưng hãy kiểm tra residual khắp ảnh và xác nhận lại calibration sau khi đổi focus, resolution hoặc cách gắn camera.

## 2. Stereo depth

Với cặp stereo đã rectified, baseline B, focal length f và disparity ngang $d=u_L-u_R$, depth gần đúng là:

$$
Z = \frac{fB}{d}
$$

Depth trở nên bất ổn khi disparity tiến gần 0. Nếu pixel disparity có sai số $\sigma_d$, uncertainty bậc nhất tăng xấp xỉ $\sigma_Z \approx (fB/d^2)\sigma_d$. Đây là lý do stereo depth xa kém chính xác. Rectification, capture đồng bộ, baseline đã biết và matching tốt đều thiết yếu.

## 3. Point cloud và rigid transform

Point cloud gồm các điểm $p_i\in\mathbb{R}^3$, thường kèm màu hoặc intensity. Đổi điểm từ camera sang robot coordinate bằng rigid transform $T=[R\;t;0\;1]$ trong homogeneous coordinate. Kiểm tra chiều của mỗi transform: “camera trong world” và “world trong camera” là hai phép nghịch đảo.

Voxel downsampling giảm mật độ; statistical/radius filtering có thể bỏ outlier nhưng cũng xóa cấu trúc mảnh. Giữ sensor timestamp và frame ID. Point cloud thiếu metadata coordinate frame/đơn vị là dữ liệu nhập nhằng.

## 4. Point-cloud registration và ICP

Iterative Closest Point (ICP) luân phiên tìm correspondence và ước lượng transform để giảm lỗi alignment. Point-to-point ICP tối thiểu hóa bình phương khoảng cách; point-to-plane thường hội tụ nhanh hơn khi normal đáng tin cậy.

ICP là phương pháp cục bộ: cần pose khởi tạo tương đối tốt và hình học có overlap. Nó có thể ghép nhầm cấu trúc lặp, thất bại khi motion lớn hoặc bị object chuyển động làm méo. Dùng robust loss, giới hạn correspondence distance, xử lý outlier và kiểm tra hội tụ. So alignment với motion đã biết hoặc sensor độc lập.

## 5. Visual odometry và SLAM

Visual odometry ước lượng motion giữa các ảnh liên tiếp. Pipeline thường phát hiện/theo dõi feature, loại outlier bằng kiểm tra hình học, ước lượng relative pose rồi tích lũy motion. Monocular scale thường nhập nhằng nếu thiếu thông tin bổ sung; stereo/depth sensor cho metric scale.

SLAM đồng thời ước lượng trajectory robot và map. Loop closure nhận ra nơi đã ghé trước đó rồi thêm constraint để sửa drift tích lũy. Factor graph biểu diễn pose và measurement thành variable/constraint, sau đó tối ưu tính nhất quán. Hệ thống thực thường hợp nhất camera, IMU, wheel odometry, GNSS hoặc lidar; mỗi sensor có failure mode riêng.

## 6. Coordinate frame và thời gian

Dùng transform tree với frame có tên như map, odom, base_link, camera_link và camera_optical_frame. Ghi handedness, chiều trục, đơn vị và quy ước transform. Matrix đúng nhưng áp sai chiều vẫn tạo motion trông có vẻ hợp lý nhưng thực tế sai.

Sensor cần được đồng bộ. Timestamp lệch giữa camera và IMU/wheel sensor khi đang di chuyển có thể trông giống lỗi calibration. Với rolling-shutter camera, các hàng ảnh được chụp ở thời điểm khác nhau nên chuyển động nhanh vi phạm giả định global shutter.

## 7. Đánh giá robotics perception trong ngữ cảnh

Đo trajectory error (absolute và relative), drift theo quãng đường/thời gian, độ nhất quán map, depth error theo range và thành công cấp tác vụ. Báo split theo sequence: frame từ một trajectory không được chia ngẫu nhiên giữa train và test. Đánh giá trên motion blur, thiếu sáng, bề mặt phản xạ, object động và sensor dropout.

Latency quan trọng: perception result đến sau khi robot đã di chuyển có thể kém hữu ích hơn kết quả hơi nhiễu hơn nhưng đến đúng lúc. Theo dõi tuổi timestamp end-to-end, không chỉ inference time neural network.

## 8. Bài tập

1. Với $f=700$ pixel, $B=0.12$ m, $d=42$ pixel, ước lượng depth. Nếu disparity giảm một nửa thì độ nhạy thay đổi ra sao?
2. ICP hội tụ với residual thấp nhưng pose robot rõ ràng sai trong hành lang có các cửa lặp lại. Vì sao?
3. Camera/IMU calibration chỉ kém khi quay nhanh. Cần kiểm tra vấn đề timing và rolling shutter nào?
4. Vì sao không phù hợp random frame split cho SLAM benchmark?

## 9. Tự kiểm tra

1. $Z=700(0.12)/42=2$ m. Disparity giảm một nửa thì depth ước lượng tăng gấp đôi; disparity error cũng gây depth error lớn hơn vì độ nhạy tỷ lệ $1/d^2$.
2. Cấu trúc lặp tạo correspondence sai và local minimum; residual thấp không chứng minh pose toàn cục đúng.
3. Kiểm tra timestamp offset, đồng bộ, IMU integration, rolling shutter và calibration residual phụ thuộc motion.
4. Frame lân cận tương quan cao; model/tuning có thể thấy gần như cùng cảnh ở cả hai partition. Split theo trajectory hoặc môi trường.
