# Workflow vision với ROS 2

[Xem bài gốc bằng tiếng Anh](../../../../curriculum/07-3d-vision-robotics/03-ros2-vision-workflow.md)

## Mục tiêu học tập

Theo dõi ảnh từ ROS 2 camera topic qua processing node, giữ timestamp/frame và kiểm tra pipeline đang chạy.

## 1. Cơ bản về ROS 2 graph

**Node** ROS 2 là process/component có trách nhiệm tập trung. Node trao đổi **message** trên **topic** có tên: publisher gửi, subscriber nhận. Message vision phổ biến gồm sensor_msgs/Image, sensor_msgs/CameraInfo và sensor_msgs/PointCloud2. Service và action dùng cho tác vụ request/response hoặc chạy lâu.

Mỗi sensor message có timestamp và thường có frame_id. Các field này thuộc ý nghĩa dữ liệu; đừng bỏ chúng chỉ để code chạy được.

## 2. Khám phá và kiểm tra topic

Khi robot hoặc bag replay đang chạy, kiểm tra graph:

~~~bash
ros2 topic list
ros2 topic info /camera/image_raw
ros2 topic hz /camera/image_raw
ros2 topic echo /camera/camera_info --once
~~~

Thay tên topic theo hệ thống của bạn. Kiểm tra message type, publishing rate, frame ID, kích thước ảnh, encoding và QoS compatibility. Sensor publisher thường dùng best-effort QoS để tránh bị consumer chậm chặn lại; subscriber reliability không tương thích có thể không nhận gì.

~~~mermaid
flowchart LR
    Camera[Camera driver] -->|Image + timestamp| Vision[Vision node]
    Info[CameraInfo + calibration] --> Vision
    TF[Transforms /tf] --> Vision
    Vision -->|Image, detection, or 3D pose| Consumer[Visualizer or planner]
~~~

## 3. Chuyển đổi và xử lý ảnh

cv_bridge đổi ROS image message thành OpenCV array và ngược lại. Một phần callback tạo grayscale/edge topic:

~~~python
from cv_bridge import CvBridge
import cv2

bridge = CvBridge()

def image_callback(msg):
    image_bgr = bridge.imgmsg_to_cv2(msg, desired_encoding="bgr8")
    gray = cv2.cvtColor(image_bgr, cv2.COLOR_BGR2GRAY)
    edges = cv2.Canny(gray, 60, 140)
    output_msg = bridge.cv2_to_imgmsg(edges, encoding="mono8")
    output_msg.header = msg.header  # preserve time and frame for same-sized pixel data
    publisher.publish(output_msg)
~~~

Đây chỉ là callback; node đầy đủ cần tạo publisher/subscriber và xử lý shutdown, exception. Nếu thuật toán crop, resize, rectify hoặc đổi camera geometry, hãy cập nhật calibration/metadata tương ứng thay vì copy CameraInfo một cách máy móc.

## 4. Ghi, phát lại và trực quan hóa

Ghi sensor topic vào ROS bag để replay thí nghiệm nhất quán:

~~~bash
ros2 bag record /camera/image_raw /camera/camera_info /tf /tf_static
ros2 bag play BAG_FOLDER
~~~

Thay BAG_FOLDER bằng path tới bag đã ghi. Trực quan hóa raw và processed topic bằng image viewer. So timestamp và rate. Replay bag hữu ích khi lặp thử nhưng không tái tạo mọi timing/hardware condition trực tiếp.

## 5. Debug từ input đến output

Kiểm tra từng ranh giới: camera publish → subscriber nhận → conversion thành công → processing tạo shape/type đúng → publisher gửi → viewer hiển thị. Log summary có throttle như kích thước, encoding và thời gian xử lý; tránh dump toàn bộ image array. Theo dõi queue depth, message drop và end-to-end delay.

## Thực hành

Chạy camera hoặc bag publisher, kiểm tra image và camera-info topic rồi tạo edge-image output. Xác minh output dimension, encoding, timestamp và frame ID. So processing delay với chu kỳ frame input. Thử nghiệm đầu tiên làm offline hoặc simulation.

## Lỗi thường gặp

- Debug computer vision trước khi xác nhận topic type, encoding và QoS.
- Copy frame ID hoặc calibration sau resize/crop mà không cập nhật geometry.
- Xử lý frame cũ đang nằm trong queue nhưng tưởng là dữ liệu hiện tại.
- Chỉ test trực tiếp nên không replay được lỗi.
