# Đánh giá và phân tích lỗi

[Xem bài gốc bằng tiếng Anh](../../../../curriculum/04-machine-learning/02-evaluation-and-error-analysis.md)

## Mục tiêu học tập

Chọn metric phù hợp với tác vụ, dùng split đúng cách và kiểm tra model thất bại ở đâu, như thế nào.

## 1. Metric cho classification

Với binary classifier:

- **True positive (TP):** trường hợp dương được dự đoán dương.
- **False positive (FP):** trường hợp âm bị dự đoán dương.
- **False negative (FN):** trường hợp dương bị dự đoán âm.
- **True negative (TN):** trường hợp âm được dự đoán âm.

Các metric chính:

$$
\operatorname{Precision}=\frac{TP}{TP+FP},
\qquad
\operatorname{Recall}=\frac{TP}{TP+FN},
\qquad
F_1=2\frac{\operatorname{Precision}\cdot\operatorname{Recall}}
{\operatorname{Precision}+\operatorname{Recall}}.
$$

Precision trả lời: “Trong các trường hợp dự đoán dương, bao nhiêu trường hợp đúng?” Recall trả lời: “Trong các ca dương thực tế, ta tìm được bao nhiêu?” $F_1$ cân bằng hai metric nhưng có thể che giấu chi phí khác nhau của từng loại lỗi. Accuracy là:

$$
\operatorname{Accuracy}=\frac{TP+TN}{TP+TN+FP+FN}.
$$

Accuracy có thể trông cao khi một class chiếm đa số.

ROC-AUC đo thứ hạng qua nhiều threshold; PR curve thường cung cấp thêm thông tin khi positive hiếm. AUC không cho biết nên triển khai threshold nào.

## 2. Metric cho regression

Với prediction số $\hat y_i$ và target $y_i$:

$$
\operatorname{MAE}=\frac{1}{n}\sum_{i=1}^{n}|y_i-\hat y_i|,
\qquad
\operatorname{MSE}=\frac{1}{n}\sum_{i=1}^{n}(y_i-\hat y_i)^2,
\qquad
\operatorname{RMSE}=\sqrt{\operatorname{MSE}}.
$$

MAE có cùng đơn vị với target và ít bị lỗi lớn chi phối. MSE nhấn mạnh lỗi lớn và có đơn vị bình phương. RMSE trở lại cùng đơn vị với target nhưng vẫn nhấn mạnh lỗi lớn.

Luôn so sánh với baseline và kiểm tra residual theo khoảng giá trị hoặc group liên quan. Error trung bình tốt có thể che lỗi ở một subgroup.

## 3. Threshold và chi phí

Classification model có thể tạo score hoặc probability. Threshold quyết định đánh đổi precision/recall. Chọn threshold trên validation data dựa trên chi phí thực tế của false positive và false negative. Threshold 0.5 có thể bỏ lỡ nhiều quyết định hợp lệ nếu class hoặc chi phí bất đối xứng.

## 4. Chiến lược split và cross-validation

Dùng test set một lần để ước lượng cuối sau khi chọn model. Cross-validation lặp lại chia train/validation để so sánh setting vững hơn khi dữ liệu ít. Giữ nguyên group và thứ tự thời gian nếu cần. Nếu label mất cân bằng, stratify khi phù hợp deployment.

## 5. Error analysis

Đừng dừng ở một score. Kiểm tra false positive/negative, pattern residual, confidence, chất lượng dữ liệu và hiệu năng theo subgroup có ý nghĩa. Gán nhóm nguyên nhân lỗi như label nhập nhằng, thiếu ngữ cảnh, pattern hiếm, distribution shift hoặc giới hạn model. Dùng nhóm lỗi để đề xuất thí nghiệm tiếp theo.

## Ví dụ có hướng dẫn

Alarm classifier có 20 alarm thật và 80 trường hợp không có alarm. Model phát hiện 15 alarm thật nhưng cũng báo nhầm 10 lần:

$$
\operatorname{Precision}=\frac{15}{15+10}=0.60,
\qquad
\operatorname{Recall}=\frac{15}{15+5}=0.75,
\qquad
\operatorname{Accuracy}=\frac{15+70}{100}=0.85.
$$

Nếu bỏ sót alarm gây tốn kém, recall 0.75 có thể quan trọng hơn accuracy. Nếu phải điều tra alarm cũng tốn kém thì false positive cũng cần xét.

## Bài tập kèm đáp án

Nếu $TP=8$, $FP=2$ và $FN=4$, precision là $8/10=0.8$, recall là $8/12\approx0.67$ và $F_1\approx0.73$. Thay đổi threshold ra sao có thể tăng recall và phát sinh chi phí nào? Hạ threshold thường bắt được nhiều positive hơn nhưng cũng tạo nhiều false positive hơn.

## Lỗi thường gặp

- Báo một metric duy nhất mà thiếu count, threshold hoặc đối chiếu.
- Tính metric trên training data rồi xem đó là generalization.
- Liên tục xem test set trong lúc tuning.
- Lấy trung bình làm mờ hiệu năng không đồng đều giữa group hoặc điều kiện vận hành.
