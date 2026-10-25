# Detection, segmentation và phân tích lỗi vision

[Xem bài gốc bằng tiếng Anh](../../../../curriculum/06-computer-vision/04-detection-segmentation-and-error-analysis.md)

Tác vụ computer vision khác nhau ở thứ chúng dự đoán và cách đo thành công. Chọn sai target format hoặc metric có thể làm kết quả nhìn mạnh nhưng vô nghĩa.

## 1. Định nghĩa task và label

- **Image classification:** dự đoán một hay nhiều label cho ảnh.
- **Object detection:** dự đoán class và bounding box cho từng object.
- **Semantic segmentation:** gán class từng pixel; các instance cùng class bị gộp.
- **Instance segmentation:** gán mask riêng cho từng object.
- **Keypoint estimation:** dự đoán điểm có cấu trúc như khớp hay landmark.

Viết hướng dẫn annotation trước khi gán nhãn: định nghĩa class, quy tắc occlusion, kích thước object tối thiểu, object bị cắt, ví dụ mơ hồ và cách biểu diễn “không biết”. Đo sự đồng thuận annotator và review bất đồng. Label không nhất quán đặt trần chất lượng model.

## 2. Bounding box và detection metric

Biểu diễn box trục song song dạng $(x_{min},y_{min},x_{max},y_{max})$ hoặc $(x_c,y_c,w,h)$ và ghi rõ quy ước tọa độ. Chỉ normalize theo kích thước ảnh nếu model/data pipeline yêu cầu tọa độ chuẩn hóa.

Intersection over Union (IoU) của box dự đoán $B_p$ và box thật $B_t$:

$$
\operatorname{IoU}(B_p,B_t)=\frac{|B_p\cap B_t|}{|B_p\cup B_t|}
$$

Detection dự đoán thường được ghép ground-truth object nếu class trùng và IoU vượt threshold. Việc ghép one-to-one: prediction trùng cùng object trở thành false positive. Average precision tóm tắt precision-recall; mAP lấy trung bình theo class và theo nhiều IoU threshold trong một số quy ước. Luôn nêu định nghĩa benchmark chính xác.

## 3. Segmentation metric

Với một class, intersection-over-union là $TP/(TP+FP+FN)$. Mean IoU lấy trung bình IoU theo class; quyết định rõ có tính class vắng mặt không. Dice/F1 là $2TP/(2TP+FP+FN)$ và liên hệ với IoU bằng $Dice=2IoU/(1+IoU)$ cho cùng binary mask.

Pixel accuracy có thể bị background chi phối. Báo per-class score, macro và frequency-weighted average khi hữu ích, cùng confusion matrix. Với tác vụ nhạy boundary, thêm boundary metric; hai mask IoU gần nhau vẫn có thể có chất lượng biên rất khác.

## 4. Data pipeline và augmentation

Ảnh và spatial annotation phải nhận cùng geometric transform. Nếu flip ảnh, cũng xoay box/mask; nếu resize, cập nhật tọa độ. Dùng nearest-neighbor interpolation cho categorical mask vì bilinear tạo class ID không tồn tại. Kiểm tra color order (RGB/BGR), pixel range, orientation metadata và preprocessing train/validation.

Augmentation hữu ích có thể gồm crop, resize, flip, color jitter, blur, noise và occlusion. Giữ chúng thực tế cho deployment: flip ngang không hợp lệ khi hướng trái/phải quan trọng; crop quá mạnh có thể bỏ object nhỏ. Validation/test image không dùng stochastic augmentation dành cho training.

## 5. Class imbalance và sampling

Class hiếm có thể bị optimizer bỏ qua. Cách xử lý gồm class-weighted loss, focal loss, balanced sampling hoặc thu thập thêm ví dụ hiếm. Các cách này đổi optimization và đôi khi calibration; đánh giá trên deployment distribution đầy đủ và tune threshold bằng validation.

## 6. Error analysis

Tạo tập review gồm false positive, false negative, localization kém, bỏ object nhỏ, occlusion, ánh sáng, camera và site. Phân loại lỗi, ước lượng tần suất từng nhóm. Tạo slice hướng đích và kiểm tra fix đề xuất có làm hại slice khác không.

Kiểm tra cả chất lượng annotation lẫn mismatch deployment. Lỗi có thể do resize hỏng, class index sai, thiếu ví dụ ban đêm hoặc chính model. Trực quan hóa prediction overlay trên ảnh ở đúng resolution dùng trong evaluation.

## 7. Đánh giá vision đáng tin cậy

Split theo video, scene, patient, camera hoặc site khi frame gần nhau/ảnh liên quan có tương quan. Random image split có thể leak frame gần như giống hệt. Cố định evaluation resolution, suppression setting, confidence threshold và model checkpoint. Nếu đích là edge/robotic deployment, đo latency và memory cùng quality.

## 8. Bài tập

1. Detector recall cao nhưng precision thấp. Cần kiểm tra pattern lỗi và operating setting nào?
2. Segmentation mask được resize bằng bilinear rồi có fractional class ID. Sai ở đâu?
3. Dataset video có hàng nghìn frame kế nhau. Vì sao random frame split lạc quan?
4. mIoU tổng tăng nhưng class quan trọng về an toàn giảm. Nên trình bày và xử lý kết quả thế nào?
