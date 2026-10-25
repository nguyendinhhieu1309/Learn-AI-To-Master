# 3D Vision và Robotics

[Xem trang gốc bằng tiếng Anh](../../../../curriculum/07-3d-vision-robotics/README.md)

Module này nối dữ liệu nhìn thấy với hình học và perception trên robot. Bắt đầu từ camera geometry trước khi hợp nhất sensor hay gắn model vào workflow robot.

## Kết quả học tập

- Biến đổi point và pose giữa coordinate frame.
- Hiểu camera calibration, depth, point cloud và tái dựng hình học.
- Thiết kế perception pipeline cơ bản giao tiếp với hệ thống robot.

## Điều kiện tiên quyết

Python, computer vision cơ bản và linear algebra. Biết robotics/ROS 2 hữu ích nhưng các khái niệm cần thiết sẽ được giới thiệu.

## Thứ tự học gợi ý

1. [Coordinate frame và camera geometry](01-coordinate-frames-and-camera-geometry.md)
2. [RGB-D, point cloud và robot pipeline](02-rgbd-point-clouds-and-robot-pipeline.md)
3. [Workflow vision với ROS 2](03-ros2-vision-workflow.md)
4. [Stereo, visual odometry, SLAM và point cloud](04-stereo-visual-odometry-slam-and-point-clouds.md)
5. [Xử lý point cloud và deep learning](05-point-cloud-processing-and-deep-learning.md)
6. [Lab point-cloud registration](06-point-cloud-registration-lab.md)

## Thực hành

- Chiếu điểm 3D lên ảnh bằng camera model đã calibration.
- Đổi điểm từ camera frame sang robot/world frame, kiểm tra đơn vị.
- Lọc point cloud nhiễu và so trực quan trước/sau.
- Làm [registration lab](06-point-cloud-registration-lab.md), ghi source/destination frame của từng transform.
- Đo end-to-end delay và frame drop trong perception pipeline nhỏ.
- Làm [bộ bài tập 3D Vision và Robotics](PROBLEM_SET_WITH_SOLUTIONS.md), gồm transform frame, depth uncertainty, registration ambiguity và point-cloud split.

## Dự án checkpoint

Xây pipeline RGB-D hoặc point cloud có thể tái tạo, nhận diện object hoặc bề mặt đơn giản. Bao gồm sơ đồ coordinate frame, giả định sensor, calibration, latency, ví dụ lỗi và đánh giá ưu tiên simulation/an toàn.

## Lỗi thường gặp cần để ý

- Trộn coordinate frame, đơn vị hoặc quy ước row/column.
- Bỏ timestamp rồi coi stream không đồng bộ là đồng thời.
- Cho rằng depth sensor chính xác ở mọi khoảng cách/bề mặt.
- Xem perception output là lệnh robot an toàn khi chưa kiểm tra uncertainty.
