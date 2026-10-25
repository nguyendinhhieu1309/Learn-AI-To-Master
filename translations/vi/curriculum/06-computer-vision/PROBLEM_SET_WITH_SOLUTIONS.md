# Computer vision: Bộ bài tập có lời giải

[Xem bài gốc bằng tiếng Anh](../../../../curriculum/06-computer-vision/PROBLEM_SET_WITH_SOLUTIONS.md)

Luyện cách ghép representation, label, metric và split ảnh với tác vụ deployment.

## 1. Image tensor layout

Ảnh màu cao 240, rộng 320, có ba channel. Nêu shape channel-last và channel-first cho batch 16 ảnh.

### Lời giải

Channel-last: [16,240,320,3]. Channel-first: [16,3,240,320]. Kiểm tra layout framework mong đợi.

## 2. Resize và aspect ratio

Ảnh 640 × 480 resize trực tiếp thành 224 × 224. Hình học thay đổi thế nào?

### Lời giải

Ảnh 4:3 bị biến dạng thành tỷ lệ 1:1. Cách khác là giữ tỷ lệ và padding, resize rồi crop hoặc letterbox. Biến đổi box/mask nhất quán và giữ preprocessing train/inference giống nhau.

## 3. Intersection over Union của bounding box

Box A phủ x=[0,4], y=[0,4]. Box B phủ x=[2,6], y=[0,4]. Dùng tọa độ liên tục. Tính IoU.

### Lời giải

Diện tích mỗi box là 16. Intersection là 2×4=8. Union bằng 16+16-8=24. IoU là 8/24=1/3.

## 4. Non-maximum suppression

Hai box cùng class có IoU 0.82. Score là 0.91 và 0.74. NMS threshold 0.5. Box nào được giữ?

### Lời giải

Giữ box score cao hơn và suppression box thấp hơn vì overlap vượt threshold. Hành vi còn tùy class handling, score threshold, tie và hard/soft NMS.

## 5. Threshold theo chi phí

False negative tốn gấp 10 lần false positive. Giả sử xác suất calibrated và chỉ xét hai chi phí này, suy ra threshold quyết định positive.

### Lời giải

Dự đoán positive khi $C_{FP}(1-p)<C_{FN}p$, vậy $p>C_{FP}/(C_{FP}+C_{FN})$. Với $C_{FN}=10C_{FP}$, threshold là $1/11\approx0.091$. Cần validation calibration và ràng buộc vận hành.

## 6. Accuracy và recall

Test có 980 negative, 20 positive. Model dự đoán mọi ca negative. Tính accuracy và positive recall.

### Lời giải

Accuracy 98%; positive recall 0%. Dùng confusion matrix, metric theo class, precision-recall analysis và operating point được chọn theo tác vụ.

## 7. IoU và Dice cho segmentation

Với một class, TP=70, FP=10, FN=20 pixel. Tính IoU và Dice.

### Lời giải

IoU=TP/(TP+FP+FN)=70/100=0.70. Dice=2TP/(2TP+FP+FN)=140/170, khoảng 0.824.

## 8. Mean average precision

Average precision tóm tắt điều gì và vì sao detection mAP cần nêu IoU threshold cùng cách average class?

### Lời giải

Average precision tóm tắt precision-recall curve dưới matching rule đã định nghĩa. Benchmark average qua class/IoU threshold khác nhau. Score chỉ so được nếu nói rõ matching, threshold, class handling và annotation bị bỏ qua.

## 9. Split theo patient, scene hoặc camera

Dataset có frame video từ 50 địa điểm. Random split frame kề nhau vào train/validation. Rủi ro là gì?

### Lời giải

Frame near-duplicate chia sẻ background, ánh sáng và object, làm validation lạc quan. Group theo sequence/location khi đánh giá cảnh chưa thấy. Nếu deployment nhắm frame tương lai của cảnh đã biết, dùng temporal split và nêu mục tiêu đó.

## 10. Augmentation và label

Áp horizontal flip lên ảnh có label trái/phải cùng box. Label nào cần biến đổi?

### Lời giải

Mirror tọa độ x của box. Biến đổi keypoint và remap directional label nếu ý nghĩa đổi. Object identity có thể giữ nguyên; label trái/phải hoặc hướng đọc text có thể không. Augmentation hợp lệ khi sample sau biến đổi vẫn giữ task label.

## 11. Leakage preprocessing

Bạn tính normalization statistic bằng cả train và validation image. Vì sao là leakage?

### Lời giải

Validation ảnh hưởng preprocessing được fit. Chỉ tính statistic trên training rồi cố định cho validation, test, deployment. Hằng số cố định bên ngoài là lựa chọn khác cần tài liệu hóa.

## 12. Error analysis theo object size

Detector recall thấp với object nhỏ nhưng cao với object lớn. Cần kiểm tra gì?

### Lời giải

Breakdown theo kích thước object, class, occlusion, source và distance. Kiểm tra annotation, resize, feature resolution, assignment strategy, threshold và NMS. Đổi từng yếu tố và so cùng slice.

## Checklist ôn tập

- Image transform và label transform có đồng bộ không?
- Split có phản ánh patient, location, camera mới hay thời gian không?
- Lỗi theo class và object size có nhìn thấy được không?
- mAP matching rule đã được ghi chưa?
- Threshold có phản ánh calibrated risk và năng lực vận hành không?
- Normalization statistic chỉ fit trên training data không?
