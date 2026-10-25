# Probability calibration, decision threshold và độ bất định

[Xem bài gốc bằng tiếng Anh](../../../../curriculum/04-machine-learning/15-calibration-and-uncertainty.md)

Classifier có thể xếp hạng ví dụ tốt nhưng báo xác suất quá cao/thấp một cách có hệ thống. Hệ thống quyết định cũng cần threshold phản ánh chi phí lỗi. Bài này tách riêng hai câu hỏi.

## Mục tiêu học tập

- Phân biệt discrimination, calibration và quyết định phân loại.
- Tính Brier score và log loss cho prediction đơn giản.
- Chọn threshold từ chi phí lỗi tường minh.
- Fit calibrator không làm leak validation/test.
- Giải thích giả định và giới hạn của conformal interval cơ bản.

## Điều kiện tiên quyết

Hiểu logistic regression, confusion matrix, validation và xác suất. Ôn [evaluation và error analysis](02-evaluation-and-error-analysis.md), [model selection](04-generalization-validation-and-model-selection.md).

## 1. Ba tính chất khác nhau

### Discrimination

Model có xếp hạng ví dụ positive ngẫu nhiên cao hơn ví dụ negative ngẫu nhiên không? AUC tóm tắt ranking qua các threshold. AUC không nói score 0.8 có nghĩa event rate 80%.

### Calibration

Trong các ví dụ được gán xác suất gần $p$, event có xảy ra xấp xỉ tỷ lệ $p$ không?

$$
P(Y=1\mid\hat p\approx p)\approx p.
$$

Calibration là phát biểu về population. Dataset hữu hạn chỉ có thể ước lượng nó với độ bất định.

### Chất lượng quyết định

Với probability, threshold và chi phí, hành động có tạo outcome chấp nhận được không? Threshold phụ thuộc bài toán, không chỉ model.

## 2. Proper scoring rule

Với nhãn nhị phân $y\in\{0,1\}$ và xác suất $p$:

$$
\operatorname{Brier}(y,p)=(p-y)^2,
\qquad
\operatorname{LogLoss}(y,p)=-y\log p-(1-y)\log(1-p).
$$

Theo phân phối dữ liệu, cả hai đều thưởng cho xác suất trung thực trong kỳ vọng. Log loss phạt rất mạnh prediction sai nhưng tự tin; implementation số thường clip xác suất tránh đúng 0 hoặc 1.

### Ví dụ có hướng dẫn

Với outcome $(1,0,1,0)$ và prediction $(0.8,0.6,0.4,0.2)$:

$$
\operatorname{Brier}=\frac{(0.8-1)^2+(0.6-0)^2+(0.4-1)^2+(0.2-0)^2}{4}
=0.25.
$$

Mean log loss:

$$
\frac{-\log(0.8)-\log(0.4)-\log(0.4)-\log(0.8)}4
\approx0.569.
$$

Ví dụ thứ hai và thứ ba sai với confidence cao nên đóng góp phần lớn vào log loss.

## 3. Reliability diagram và expected calibration error

Để kiểm tra calibration:

1. Chia prediction thành các bin xác suất.
2. Với từng bin, vẽ mean predicted probability so với tần suất positive quan sát.
3. Hiện số ví dụ trong mỗi bin; bin chỉ có năm ví dụ sẽ nhiễu.

Expected calibration error (ECE) là trung bình có trọng số của độ chênh tuyệt đối giữa accuracy và confidence theo bin. ECE phụ thuộc cách chia bin và không thay proper scoring rule. Báo phương pháp chia bin, dùng reliability plot cùng Brier score hoặc log loss.

## 4. Phương pháp calibration

- **Platt scaling:** fit logistic mapping từ model score ngoài mẫu sang label quan sát. Mượt và gọn.
- **Isotonic regression:** fit mapping không giảm mà không giả định hình sigmoid. Linh hoạt nhưng cần nhiều calibration data để tránh overfit.
- **Temperature scaling:** điều chỉnh scale của logit nhưng thường giữ nguyên ranking class trong multiclass.

Calibration data phải tách khỏi data fit base model, hoặc tạo bằng cross-validation sao cho prediction out-of-fold. Không fit calibrator lên prediction in-sample rồi tuyên bố ước lượng calibration tương lai.

## 5. Suy ra threshold theo chi phí

Giả sử action 1 nghĩa là “can thiệp”. Gọi $C_{FP}$ là chi phí can thiệp vào ca âm, $C_{FN}$ là chi phí không can thiệp ca dương. Với xác suất calibrated $p$:

- expected cost của action 1: $(1-p)C_{FP}$;
- expected cost của action 0: $pC_{FN}$.

Chọn action 1 khi:

$$
(1-p)C_{FP}<pC_{FN}
\quad\Longleftrightarrow\quad
p>\frac{C_{FP}}{C_{FP}+C_{FN}}.
$$

Nếu false negative tốn 4 đơn vị và false positive tốn 1, threshold là $1/(1+4)=0.2$. Kết quả này giả định probability/chi phí phù hợp population deployment và action không chịu constraint bổ sung.

## 6. Prior shift và giám sát

Nếu prevalence class đổi nhưng phân phối score theo class giữ ổn định, calibration có thể drift dù ranking tương tự. Trong production, theo dõi:

- calibration theo time window và subgroup quan trọng;
- Brier hoặc log loss kèm confidence interval;
- thay đổi prevalence và missingness;
- volume action tại threshold đã chọn;
- độ trưởng thành của label đến trễ.

Recalibration phải dùng label đã trưởng thành và quy trình tôn trọng thời gian. Không suy ra performance chỉ từ traffic chưa có label.

## 7. Prediction interval bằng split conformal regression

Split conformal prediction tạo interval đơn giản dưới giả định exchangeability:

1. Fit regression model trên training set.
2. Trên calibration set riêng, tính residual score $s_i=|y_i-\hat y_i|$.
3. Với miscoverage mục tiêu $\alpha$, chọn empirical quantile hiệu chỉnh theo rank $\lceil(n_{\mathrm{cal}}+1)(1-\alpha)\rceil$.
4. Trả về $[\hat y(x)-q,\hat y(x)+q]$.

Với exchangeability và split procedure đã nêu, guarantee là marginal coverage trên ví dụ tương lai, không phải coverage có điều kiện chính xác cho mọi subgroup hay mọi $x$. Time dependence, distribution shift và leakage có thể làm mất guarantee. Interval có thể rộng; chỉ có coverage chưa đủ để hữu ích.

## Thực hành

1. Hai model có cùng AUC nhưng một model log loss tệ hơn. Điều gì có thể khác?
2. Model dự đoán xác suất 0.8 cho 50 ví dụ validation tương tự, trong đó có 30 positive. Bin đó có calibrated không?
3. Tính threshold khi false positive cost bằng 3 và false negative cost bằng 2.
4. Vì sao calibration trên prediction training thường lạc quan?
5. Interval conformal 90% bảo đảm điều gì và không bảo đảm điều gì?

### Gợi ý đáp án

1. Ranking có thể tương tự nhưng một model gán probability quá tự tin hoặc thiếu tự tin.
2. Tỷ lệ quan sát là $30/50=0.6$, thấp hơn xác suất dự đoán 0.8; bin gợi ý model quá tự tin, nhưng cần xét sampling uncertainty.
3. $C_{FP}/(C_{FP}+C_{FN})=3/5=0.6$.
4. Base model đã fit chính các label đó nên score không đại diện prediction thật sự chưa thấy.
5. Với exchangeability và cách tách đúng, mục tiêu là marginal coverage xấp xỉ 90% trên label tương lai. Không bảo đảm coverage 90% ở mọi subgroup hay khi phân phối tùy ý thay đổi.

## Lỗi thường gặp

- Gọi AUC là “confidence”.
- Đổi threshold mà không nêu chi phí hay ràng buộc vận hành.
- Dùng cùng dữ liệu để train, calibrate, tune và báo performance cuối.
- Xem ECE độc lập với cách chia bin.
- Tuyên bố conformal guarantee sau time leakage hoặc population shift.
