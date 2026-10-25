# RGB-D, point cloud và pipeline robot

[Xem bài gốc bằng tiếng Anh](../../../../curriculum/07-3d-vision-robotics/02-rgbd-point-clouds-and-robot-pipeline.md)

## Mục tiêu học tập

Liên hệ depth image với điểm 3D, nhận biết thao tác point cloud thường dùng và thiết kế đường perception đáng tin cậy đưa vào hệ thống phần mềm robot.

## 1. Depth image

RGB-D camera ghép màu với phép đo depth mỗi pixel (hoặc các pixel gần alignment). Với pixel (u,v) và depth Z, đảo pinhole equation:

$$
X=(u-c_x)Z/f_x,\qquad Y=(v-c_y)Z/f_y.
$$

Quy ước depth có thể là depth theo trục quang học hoặc khoảng cách dọc tia; đọc tài liệu sensor. Depth image thường có giá trị không hợp lệ (zero, NaN hoặc sentinel), kém tin cậy trên bề mặt bóng, trong suốt, tối hoặc xa.

## 2. Point cloud

Point cloud là tập điểm 3D, đôi khi kèm color, normal hoặc confidence. Thao tác phổ biến:

- **Voxel downsampling:** thay các điểm gần nhau trong một ô lưới 3D bằng điểm đại diện.
- **Outlier filtering:** bỏ điểm cô lập nhưng tránh xóa cấu trúc thưa hợp lệ.
- **Normal estimation:** ước lượng hướng bề mặt cục bộ, hữu ích cho ghép hình học.
- **Registration:** căn chỉnh hai cloud, thường bắt đầu từ pose gần đúng rồi tinh chỉnh bằng phương pháp như ICP.

Các bước này có thể thất bại khi overlap ít, hình học lặp, có chuyển động hoặc initial alignment kém. Trực quan hóa trước/sau mỗi bước và theo dõi coordinate frame.

## 3. Đồng bộ sensor và khái niệm ROS 2

Trong robot pipeline, image và depth message cần timestamp tương thích, frame ID và calibration. Transform tree lưu quan hệ giữa frame. Perception node cần khai báo input topic, output type, đơn vị, frame, timestamp và quality/confidence. Dùng QoS profile phù hợp và giám sát message rơi/trễ; thuật toán đúng trên dữ liệu cũ vẫn có thể làm robot hành động sai.

## 4. Từ perception đến action

Object detection trong pixel chưa phải robot pose. Chuyển pixel/depth sang camera-frame coordinate, đổi sang robot/world coordinate, kiểm tra confidence và workspace hợp lệ, rồi gửi kết quả có timestamp cho planning. Giữ safety check và motion constraint trong robot-control layer. Bắt đầu bằng simulation hoặc offline playback trước khi tác động vật lý.

Đo end-to-end latency: sensor capture → transport → synchronization → preprocessing → inference → transform → decision. Thời gian trung bình có thể che delay đuôi dài; đo percentile và frame-drop rate.

## Bài tập nhỏ

Cho fx=500, cx=320, pixel u=370 và depth Z=2 m, tọa độ ngang là $(370-320)*2/500=0.2$ m. Trước khi dùng hãy hỏi: depth có đơn vị mét không, pixel 370 có thuộc ảnh đã calibration không và điểm đầu ra nằm trong frame nào?

## Dự án checkpoint

Xây pipeline từ data ghi sẵn hoặc simulation tạo một điểm 3D/object pose. Bao gồm sơ đồ frame, giả định calibration/timestamp, xử lý depth không hợp lệ, latency statistic, ví dụ lỗi và ranh giới an toàn không kích hoạt cơ cấu chấp hành.

## Lỗi thường gặp

- Giả định RGB/depth đã alignment hoặc đồng bộ.
- Bỏ frame ID rồi đoán hệ tọa độ về sau.
- Coi mọi depth pixel đều hợp lệ.
- Gửi perception thẳng tới motion control mà không kiểm tra workspace/confidence.
