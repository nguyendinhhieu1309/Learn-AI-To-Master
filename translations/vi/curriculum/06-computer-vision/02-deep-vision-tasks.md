# Tác vụ deep vision

[Xem bài gốc bằng tiếng Anh](../../../../curriculum/06-computer-vision/02-deep-vision-tasks.md)

## Mục tiêu học tập

Phân biệt classification, detection và segmentation; hiểu metric mỗi tác vụ dùng và lập kế hoạch error analysis cho visual label.

## 1. Classification

Image classification trả về một hoặc nhiều category score cho toàn ảnh. Single-label task chọn một trong các class loại trừ lẫn nhau; multi-label task có thể gán nhiều nhãn cùng lúc. Class balance và label mơ hồ đều quan trọng. Kiểm tra lỗi theo category, chất lượng ảnh, ánh sáng và nguồn thu thập.

## 2. Object detection

Detection dự đoán class và bounding box của object. Box thường có dạng [x_min,y_min,x_max,y_max]. **Intersection over Union (IoU)** là:

$$
IoU = \frac{\text{diện tích giao}}{\text{diện tích hợp}}.
$$

IoU threshold quyết định predicted box có khớp ground-truth box không. Non-maximum suppression (NMS) bỏ dự đoán trùng lặp chồng lấn. Average precision tóm tắt precision-recall curve; mean average precision (mAP) lấy trung bình AP qua class và đôi khi qua IoU threshold. Luôn nêu quy ước evaluation vì các biến thể mAP khác nhau.

## 3. Segmentation

Semantic segmentation gán class cho từng pixel. Instance segmentation còn phân biệt các object riêng có cùng class. Mean IoU lấy trung bình overlap theo class. Accuracy cấp ảnh cao vẫn có thể đi cùng boundary kém hoặc thất bại trên object nhỏ; hãy xem mask trực quan và báo kết quả từng class.

## 4. Data và label

Ảnh từ cùng video, site hoặc người thường nên giữ trong cùng split. Nếu không near-duplicate có thể leak. Quy tắc gán nhãn cần định nghĩa occlusion, truncation, class mơ hồ và object nhỏ. Với label chủ quan, đo mức đồng thuận annotation. Tài liệu hóa resolution, camera, ánh sáng và quyền dùng data.

## 5. Augmentation và deployment

Augmentation có thể tăng robustness nếu giống biến thiên hợp lệ: đổi sáng nhẹ, crop, rotation hoặc noise. Nó không thay thế dữ liệu evaluation thực đại diện. Khi deploy, xác minh channel order, normalization, resize và threshold khớp training. Đo latency và memory trên thiết bị đích.

Xem khác biệt deployment là một phần bài toán vision: rolling shutter, motion blur, thiếu sáng, compression, glare và vị trí camera thay đổi đều có thể shift phân phối input. Đo rõ điều kiện liên quan.

## Thực hành

Với detector, chọn 20 false positive và 20 false negative. Phân nhóm theo nguyên nhân như object rất nhỏ, blur, occlusion, background gây nhầm hoặc annotation sai. Chọn cải tiến và nêu cách đo ích lợi mà không dùng lại test set để tune.

## Lỗi thường gặp

- So detection score mà không nêu IoU và cách lấy trung bình class.
- Đưa frame video gần trùng vào cả train lẫn test.
- Chỉ báo mAP tổng, bỏ qua lỗi theo class.
- Inference resize/normalize khác training.
