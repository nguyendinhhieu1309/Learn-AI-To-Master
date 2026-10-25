# Hệ tọa độ và hình học camera

[Xem bài gốc bằng tiếng Anh](../../../../curriculum/07-3d-vision-robotics/01-coordinate-frames-and-camera-geometry.md)

## Mục tiêu học tập

Mô tả một điểm 3D trong hệ tọa độ, biến đổi điểm giữa các frame và chiếu điểm lên ảnh bằng pinhole camera model.

## 1. Tọa độ không có nghĩa nếu thiếu frame

Điểm (1, 0, 0) có thể là một mét phía trước camera, robot base hoặc gốc world. Frame xác định origin và hướng các trục. Đặt tên frame tường minh: p_camera, p_base, p_world. Nêu đơn vị (mét hay millimeter) cùng quy ước right-handed hay left-handed.

## 2. Rigid transform

Rigid transform quay và tịnh tiến điểm mà không làm đổi hình dạng. Dùng homogeneous coordinate:

$$
\begin{bmatrix}p_A\\1\end{bmatrix} =
T_{A\leftarrow B}\begin{bmatrix}p_B\\1\end{bmatrix}, \qquad
T = \begin{bmatrix}R&t\\0&1\end{bmatrix}.
$$

$T_{A\leftarrow B}$ nghĩa là “đổi tọa độ biểu diễn trong frame B sang frame A”. Rotation R là 3×3, translation t là 3×1. Ghép transform theo thứ tự:

$$
T_{A\leftarrow C}=T_{A\leftarrow B}T_{B\leftarrow C}.
$$

Phép nhân matrix không giao hoán; đảo thứ tự transform làm đổi kết quả.

## 3. Pinhole camera projection

Với điểm trong camera frame (X,Y,Z), Z>0, tọa độ chuẩn hóa là x=X/Z, y=Y/Z. Tọa độ pixel xấp xỉ:

$$
u=f_xx+c_x,\qquad v=f_yy+c_y.
$$

$f_x,f_y$ là focal length theo pixel; $(c_x,c_y)$ là principal point. Dạng matrix: $s[u,v,1]^T=K[X,Y,Z]^T$, với K là intrinsic matrix. Lens distortion làm cong phép chiếu lý tưởng; calibration ước lượng intrinsic và distortion parameter cho cấu hình camera/focus/resolution cụ thể.

## 4. Calibration và uncertainty

Calibration ước lượng cách camera ánh xạ hình học 3D đã biết lên điểm ảnh. Dùng calibration target được quan sát rõ ở nhiều vị trí và hướng. Xem reprojection error nhưng nhớ rằng trung bình thấp có thể che lỗi góc ảnh hoặc resolution không khớp. Calibrate lại sau khi đổi lens/focus hay hình học camera.

Với nhiều sensor, depth uncertainty tăng theo khoảng cách. Sai số depth hoặc camera pose có thể làm điểm chiếu lệch đáng kể. Hãy mang uncertainty qua pipeline nếu quyết định phụ thuộc vào vị trí chính xác.

### Ví dụ đổi frame

Nếu camera tịnh tiến 0.2 m về phía trước robot base mà không quay, điểm ở camera coordinate (0,0,1) có base coordinate (0.2,0,1) theo quy ước nêu trên. Định nghĩa trục khác sẽ đổi dấu/hướng. Hãy vẽ frame trước khi viết matrix.

## Ví dụ chiếu điểm nhỏ

~~~python
import numpy as np

fx, fy = 500.0, 500.0
cx, cy = 320.0, 240.0
X, Y, Z = 0.2, -0.1, 2.0  # meters in the camera frame; Z must be positive
u = fx * X / Z + cx
v = fy * Y / Z + cy
print(u, v)  # (370.0, 215.0) pixels
~~~

Phép tính pinhole lý tưởng này bỏ qua lens distortion. Trước khi so với ảnh thật, xác nhận calibration thuộc đúng camera/resolution và điểm được biểu diễn trong camera optical frame.

## Thực hành

Vẽ trục camera và robot base. Viết bằng lời chiều transform, ghép hai transform và kiểm tra bằng điểm dễ như origin. Kiểm tra projection: nếu X=0, điểm nằm trên đường giữa dọc ảnh $u=c_x$ (bỏ qua distortion).

## Lỗi thường gặp

- Dùng transform sai chiều hoặc nhân sai thứ tự.
- Trộn degree với radian, millimeter với meter hoặc quy ước handedness.
- Coi tọa độ pixel là tọa độ metric.
- Dùng calibration của resolution này cho resolution khác mà không scale/kiểm tra intrinsic.
