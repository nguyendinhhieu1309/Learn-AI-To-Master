# Generalization, validation và model selection

[Xem bài gốc bằng tiếng Anh](../../../../curriculum/04-machine-learning/04-generalization-validation-and-model-selection.md)

Training model dễ hơn nhiều so với đo xem nó có hoạt động trên dữ liệu mới không. Bài này giải thích quy trình đánh giá và các cách thường khiến kết quả sai lệch.

## 1. Generalization và chia dữ liệu

Gọi $R(f)$ là loss kỳ vọng của model $f$ trên phân phối deployment thực tế; $\hat{R}(f)$ là loss trung bình trên dataset hữu hạn. Ta đo được $\hat{R}$, nhưng điều ta quan tâm là $R$. Chênh lệch giữa chúng là generalization error.

Dùng ba vai trò dữ liệu:

- **Training set:** fit tham số.
- **Validation set:** chọn feature, hyperparameter, threshold và checkpoint.
- **Test set:** ước lượng hiệu năng cuối sau khi đã chốt quyết định.

Nếu test result dẫn tới một quyết định khác, test set đã trở thành validation data. Hãy giữ một test set mới, chưa đụng đến, cho kết luận cuối.

Chọn split phản ánh deployment:

- Random split cho ví dụ độc lập, cùng phân phối.
- Group split khi record cùng người, thiết bị, công ty, video hay nguồn.
- Time split khi mục tiêu là dự đoán tương lai.
- Spatial/site split khi model cần chuyển sang địa điểm hoặc tổ chức mới.

## 2. Leakage: lối tắt ẩn

Leakage xảy ra khi training hoặc model selection dùng thông tin không có lúc dự đoán. Nó có thể là bản sao trực tiếp của target, feature sau outcome, record trùng giữa split, preprocessing fit trên toàn bộ data hoặc feature phụ thuộc thời gian tiết lộ tương lai.

Fit mọi transformation học được chỉ trên training data:

~~~python
pipeline = Pipeline([
    ("impute", SimpleImputer(strategy="median")),
    ("scale", StandardScaler()),
    ("model", LogisticRegression()),
])
pipeline.fit(X_train, y_train)
pred = pipeline.predict(X_valid)
~~~

Pipeline fit imputation và scaling bên trong từng training fold. Tính global mean trước cross-validation làm rò thông tin từ validation fold vào training.

Câu hỏi audit leakage:

1. Feature này có tồn tại đúng lúc đưa ra dự đoán không?
2. Một người, object hay near-duplicate có thể xuất hiện cả train và test không?
3. Label hoặc aggregate statistic có được tính từ outcome tương lai không?
4. Human review, prompt tuning hoặc threshold selection có dùng test set không?
5. Test set có đại diện cho population và thời kỳ deployment không?

## 3. Cross-validation

Trong $k$-fold cross-validation, chia training data thành $k$ fold, fit trên $k-1$ fold rồi đánh giá trên fold còn lại; lặp đến khi mỗi fold từng làm validation một lần. Lấy trung bình fold score và báo độ biến thiên, đồng thời nhớ rằng các fold không phải những thí nghiệm độc lập.

Dùng stratified fold cho classification nếu phù hợp, group-aware fold cho hàng có tương quan và time-aware validation cho forecasting. Cơ chế split là một phần của thí nghiệm và cần được ghi lại.

Nested cross-validation tách việc chọn hyperparameter khỏi việc ước lượng hiệu năng: inner fold tune model; outer fold ước lượng toàn bộ quy trình selection. Cách này tốn hơn nhưng tránh báo cáo lạc quan khi dữ liệu ít.

## 4. Bias, variance và learning curve

Training error cao thường gợi ý underfitting: model/feature không biểu diễn được pattern, optimization chưa hoàn tất hoặc regularization quá mạnh. Training error thấp nhưng validation error cao hơn nhiều có thể là overfitting hoặc sai khác phân phối/split.

Vẽ learning curve theo kích thước training set tăng dần. Nếu train và validation error đều cao và gần nhau, thêm dữ liệu cùng loại có thể không giúp nhiều. Nếu train error thấp nhưng validation cao, dữ liệu đại diện hơn, regularization mạnh hơn hoặc model đơn giản hơn có thể hữu ích.

Đây là manh mối, không phải bằng chứng. Trước tiên loại trừ bug, leakage, lỗi label và split không tương thích.

## 5. Tìm hyperparameter

- **Grid search:** tìm có hệ thống nhưng có thể phí lượt thử cho chiều không quan trọng.
- **Random search:** baseline tốt khi chỉ vài chiều quyết định.
- **Successive halving:** cấp ngân sách nhỏ cho nhiều ứng viên rồi tăng cho phương án triển vọng.
- **Bayesian optimization:** dùng surrogate chọn lượt thử tiếp theo giàu thông tin, nhất là tuning tốn kém và ít chiều.

Tune bằng validation hoặc inner cross-validation. Ghi search space, random seed, compute budget và metric chọn model. Bản thân search cũng có thể overfit, vì vậy vẫn cần giữ final test set.

## 6. Threshold và calibration

Với xác suất $p(x)$ và threshold $t$, gán positive khi $p(x)\ge t$. Threshold quyết định đánh đổi precision/recall; 0.5 không phải lựa chọn tối ưu trong mọi trường hợp. Chọn bằng validation dựa trên chi phí tương đối của false positive và false negative.

Calibration hỏi rằng những event được dự đoán xác suất $q$ có xảy ra xấp xỉ tỷ lệ $q$ hay không. Classifier đã calibrated vẫn có thể xếp hạng kém; ranker tốt cũng có thể calibrated kém. Dùng reliability plot, Brier score hoặc log loss; calibrate bằng prediction held-out, không bao giờ dùng label final test.

## 7. Chọn metric và slice

Với positive class hiếm, accuracy có thể cao dù model bỏ sót gần hết positive. Báo metric phù hợp quyết định:

- **Precision:** trong các trường hợp dự đoán positive, bao nhiêu trường hợp đúng?
- **Recall:** trong số positive thực tế, model tìm được bao nhiêu?
- **F1:** trung bình điều hòa precision và recall tại một threshold.
- **PR-AUC:** khả năng xếp hạng chú trọng positive; baseline phụ thuộc prevalence.
- **ROC-AUC:** xếp hạng cặp mẫu qua các threshold; khi mất cân bằng nặng có thể trông đáng yên tâm dù chưa đủ.
- **Log loss/Brier score:** chất lượng xác suất, kể cả confidence.

Luôn kiểm tra confusion count, threshold vận hành và hiệu năng theo lát cắt có ý nghĩa như site, ngôn ngữ, thiết bị hoặc thời gian. Slice analysis dùng để chẩn đoán; slice nhỏ có độ bất định lớn.

## 8. Ví dụ quyết định có hướng dẫn

Giả sử fraud model bắt được nhiều gian lận hơn nhưng cũng chặn nhiều giao dịch hợp pháp hơn. Trước hết chọn đơn vị và chi phí của lỗi. Dùng validation prediction vẽ số vụ gian lận phát hiện được so với số giao dịch hợp pháp bị chặn qua các threshold. Chọn điểm vận hành thỏa ràng buộc block-rate đã định trước. Ước lượng điểm đã chọn một lần trên test set chưa đụng đến, kèm confidence interval và kiểm tra theo segment.

## 9. Bài tập

1. Bạn dự đoán bệnh nhân tái nhập viện và mỗi bệnh nhân có nhiều record. Nên dùng split nào, vì sao?
2. Bước preprocessing chuẩn hóa feature bằng mean toàn dataset trước 5-fold cross-validation. Có lỗi gì và sửa ra sao?
3. Classifier phát hiện event hiếm đạt accuracy 99% nhưng recall 2%. Cần thêm metric và count nào?
4. Model thắng trên random split nhưng thua trên dữ liệu tháng sau. Điều này nói gì về thiết kế đánh giá?

## 10. Tự kiểm tra

1. Dùng split theo patient group; nếu không, pattern riêng của bệnh nhân có thể leak qua fold.
2. Đặt preprocessing trong pipeline và fit độc lập bên trong từng training fold.
3. Kiểm tra prevalence, confusion matrix, precision, PR-AUC, calibration và chi phí tại threshold ứng viên.
4. Hiệu năng random split không ước lượng tốt deployment theo thời gian. Chẩn đoán drift và validation bằng split thời gian tiến tới hoặc rolling.
