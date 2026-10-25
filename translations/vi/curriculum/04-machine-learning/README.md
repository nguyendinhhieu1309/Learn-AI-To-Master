# Machine Learning

[Xem trang gốc bằng tiếng Anh](../../../../curriculum/04-machine-learning/README.md)

## Kết quả học tập

- Chuyển câu hỏi thực tế thành target, feature set, baseline, metric và quyết định deployment.
- Suy ra objective của linear/logistic regression, nearest neighbors, k-means, SVM, tree và matrix factorization.
- Chọn split và metric phù hợp với cách dữ liệu tương lai sẽ đến.
- Chẩn đoán underfitting, overfitting, leakage, class imbalance, calibration và distribution shift.
- Giải thích giả định model cho phép học gì và prediction có thể thất bại ở đâu.

## Điều kiện tiên quyết

Python cơ bản, xử lý dữ liệu, linear algebra và xác suất nền tảng. Ôn [các chủ đề Toán](../01-math/README.md) khi cần.

## Thứ tự học gợi ý

### A. Bắt đầu từ định nghĩa bài toán

1. [Supervised learning và baseline](01-supervised-learning.md)
2. [Evaluation và error analysis](02-evaluation-and-error-analysis.md)
3. [Generalization, validation và model selection](04-generalization-validation-and-model-selection.md)

### B. Học thuật toán cốt lõi theo thứ tự

4. [Linear regression và gradient descent](07-linear-regression-and-gradient-descent.md)
5. [Logistic regression, perceptron và softmax](08-logistic-regression-perceptron-and-softmax.md)
6. [k-NN và k-means](09-knn-and-k-means.md)
7. [Decision tree và Naive Bayes](12-decision-trees-and-naive-bayes.md)
8. [Support Vector Machine](10-support-vector-machines.md)
9. [PCA, SVD và dimensionality reduction](11-pca-svd-and-dimensionality-reduction.md)
10. [Recommendation system](13-recommendation-systems.md)

### C. Mở rộng và củng cố

11. [Tổng quan unsupervised learning](03-unsupervised-learning.md)
12. [Thuật toán cổ điển và chọn model](05-classical-ml-algorithms-and-model-choice.md)
13. [Unsupervised learning và anomaly detection](06-unsupervised-learning-and-anomaly-detection.md)
14. [Ensemble và boosting](14-ensembles-and-boosting.md)
15. [Probability calibration, threshold và uncertainty](15-calibration-and-uncertainty.md)

Với mỗi thuật toán, hãy tự tái tạo ví dụ bằng tay trước, sau đó cài phiên bản nhỏ, so với thư viện tin cậy và kiểm tra lỗi trên validation split. Dùng bài [calculus và optimization](../01-math/02-calculus-and-optimization.md) nếu chưa rõ gradient step và bài [likelihood](../01-math/08-likelihood-mle-and-map.md) để hiểu dạng của loss phổ biến.

## Thực hành

- Làm [bộ bài tập ML kèm lời giải](PROBLEM_SET_WITH_SOLUTIONS.md) sau các bài thuật toán.
- Dự đoán target đơn giản bằng constant baseline trước khi train model.
- So sánh precision, recall, F1 và confusion matrix ở nhiều threshold.
- Đánh giá regression model bằng nhiều metric và kiểm tra residual.
- So sánh random validation với time-aware validation trên dataset thời gian.
- Fit linear regression bằng least squares lẫn gradient descent; kiểm tra gradient số trên dataset nhỏ.
- So sánh logistic regression với RBF SVM trên cùng dataset đã scale, rồi kiểm tra calibration và ảnh hưởng threshold.
- Chạy k-means nhiều seed và giải thích thuộc tính cluster nào ổn định.
- Tạo evaluation theo thời gian cho recommender và giải thích log interaction không cho biết điều gì.

## Dự án checkpoint

Xây dựng nghiên cứu dự đoán dạng bảng hoàn chỉnh: định nghĩa tác vụ, tài liệu hóa dataset, thiết lập baseline, so sánh ít nhất hai họ model bằng split hợp lệ, phân tích lỗi và viết kết luận ngắn kèm giới hạn.

[Cheat sheet công thức ML](../01-math/FORMULA_CHEATSHEET.md) là công cụ ôn tập, không thay thế việc tự suy ra objective trong các bài học.

## Lỗi thường gặp cần để ý

- Chọn model chỉ dựa vào training score.
- Tuning lặp lại trên test set.
- Chọn accuracy cho tác vụ mất cân bằng nặng mà không kiểm tra lỗi theo class.
- Coi feature importance là bằng chứng nguyên nhân.
