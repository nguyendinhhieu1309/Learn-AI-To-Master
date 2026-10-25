# Image và computer vision cổ điển

[Xem bài gốc bằng tiếng Anh](../../../../curriculum/06-computer-vision/01-images-and-classical-vision.md)

## Mục tiêu học tập

Đọc array ảnh, hiểu filter đơn giản làm ảnh thay đổi thế nào và dùng phép toán vision cổ điển trước khi chọn learned model.

## 1. Biểu diễn ảnh số

Grayscale image là array 2D có shape (height, width). Color image thường là (height, width, channels), với 3 channel RGB/BGR hoặc 4 channel RGBA. Giá trị có thể là integer 0–255 hoặc float trong [0,1]. Luôn kiểm tra thứ tự màu của thư viện; OpenCV thường nạp BGR, còn nhiều công cụ vẽ yêu cầu RGB.

Tọa độ ảnh thường dùng x theo cột ngang và y đi xuống theo hàng. Array indexing thường là [row, column] = [y, x]. Đây là nguồn nhầm lẫn phổ biến gây transpose tọa độ.

## 2. Color và intensity

RGB biểu diễn các channel màu cộng. HSV tách hue, saturation và value; trong vài điều kiện việc threshold theo màu trực quan hơn. Chuyển grayscale kết hợp channel nhưng làm mất thông tin màu. Chuẩn hóa và resize nhất quán giữa training và inference.

## 3. Filter và edge

Convolution filter tính neighborhood có trọng số quanh mỗi pixel. Blur lấy trung bình vùng lân cận, giảm chi tiết/noise tần số cao. Edge filter phản ứng với thay đổi intensity nhanh. Thresholding chuyển ảnh xám thành binary mask. Morphological erosion làm foreground nhỏ lại; dilation mở rộng; opening/closing kết hợp để bỏ noise nhỏ hoặc lấp khe.

Các phương pháp này nhanh, dễ diễn giải, nhưng threshold cố định có thể thất bại khi ánh sáng, exposure hay camera condition đổi.

## 4. Biến đổi hình học

Resize, crop, rotate và perspective transform làm đổi pixel coordinate. Với tác vụ supervised, hãy biến đổi label cùng ảnh: box và mask phải giữ alignment. Với classification, random crop/flip chỉ là augmentation hữu ích nếu không đổi ý nghĩa label (lật trái/phải có thể làm sai nhãn hướng).

## Ví dụ nhỏ

Ảnh 640 × 480 thường có NumPy shape (480, 640, 3). Pixel [10, 20] nghĩa là hàng 10, cột 20. Tọa độ hiển thị thường là (x=20,y=10). Nếu box lưu dạng (x_min,y_min,x_max,y_max), đừng index như thể thứ tự là (row_min,col_min,...) mà chưa đổi.

## Ví dụ OpenCV nhỏ

~~~python
import cv2

image_bgr = cv2.imread("image.jpg")
if image_bgr is None:
    raise FileNotFoundError("Could not read image.jpg")

gray = cv2.cvtColor(image_bgr, cv2.COLOR_BGR2GRAY)
blurred = cv2.GaussianBlur(gray, (5, 5), 0)
edges = cv2.Canny(blurred, threshold1=60, threshold2=140)
~~~

Threshold chỉ là giá trị ví dụ, không phổ quát. Nếu output kém, hãy xem ảnh grayscale/blur rồi tune theo điều kiện ánh sáng đại diện.

## Thực hành

Chọn ảnh, hiển thị từng channel, ảnh grayscale, blur và edge map. Ghi một chi tiết biến mất và một chi tiết rõ hơn sau mỗi transformation.

## Lỗi thường gặp

- Nhầm channel BGR với RGB.
- Nhầm width/height ảnh với row/column array.
- Resize ảnh nhưng quên scale tọa độ bounding box.
- Augmentation làm đổi nghĩa label.
