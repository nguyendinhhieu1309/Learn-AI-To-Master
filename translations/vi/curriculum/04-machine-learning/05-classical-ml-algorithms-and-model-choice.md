# Thuật toán ML cổ điển và chọn model

[Xem bài gốc bằng tiếng Anh](../../../../curriculum/04-machine-learning/05-classical-ml-algorithms-and-model-choice.md)

Không thuật toán nào thắng trên mọi dataset. Bài này giải thích supervised method phổ biến, giả định của chúng và cách so sánh công bằng.

## 1. Linear model

Linear regression dự đoán tổng có trọng số. Logistic regression đưa linear score qua sigmoid hoặc softmax để ước lượng xác suất class. Linear model nhanh, dễ diễn giải và là baseline mạnh; chúng có thể cần feature transform hoặc interaction để mô tả cấu trúc phi tuyến.

Regularization kiểm soát độ lớn coefficient. L1 có thể đưa một số coefficient về đúng 0; L2 thu nhỏ tất cả coefficient. Tune regularization trên validation data và scale feature nếu muốn penalty áp dụng công bằng lên coefficient.

## 2. Decision tree

Decision tree đệ quy chia feature space bằng split cải thiện tiêu chí impurity như Gini impurity hoặc entropy. Tree xử lý interaction phi tuyến và kiểu feature hỗn hợp với ít scaling, nhưng tree sâu có thể ghi nhớ training example. Kiểm soát depth, leaf size, pruning và xem độ ổn định khi dữ liệu thay đổi nhẹ.

## 3. Bagging và random forest

Bagging training model trên bootstrap sample rồi tổng hợp prediction để giảm variance. Random forest còn xét ngẫu nhiên một tập con feature tại mỗi split, làm giảm tương quan giữa các tree. Chúng là baseline tổng quát bền vững cho dữ liệu dạng bảng, dù category có cardinality rất cao và ngoại suy ngoài miền train có thể là điểm yếu.

## 4. Gradient-boosted tree

Boosting lần lượt thêm weak learner để sửa residual trước đó hoặc tối ưu objective khả vi. Thư viện gradient-boosted tree thường hoạt động rất tốt trên dữ liệu có cấu trúc. Setting chính gồm số tree, learning rate, depth/leaves, row/feature subsampling, regularization và early stopping.

Learning rate thấp thường cần nhiều tree hơn. Tune bằng validation split phản ánh thời gian và group. Cẩn trọng với leakage trong feature sinh từ target và encoding có cardinality cao.

## 5. k-nearest neighbors và naive Bayes

k-NN dự đoán dựa trên các ví dụ gần nhất. Phương pháp đơn giản, có thể mô hình hóa pattern cục bộ nhưng cần distance đã scale có ý nghĩa và inference có thể chậm/tốn bộ nhớ. Naive Bayes áp dụng Bayes' rule với giả định độc lập có điều kiện; dù giả định mạnh, đây vẫn là baseline văn bản hiệu quả và hoạt động tốt với count thưa.

## 6. So sánh model công bằng

Dùng cùng split dữ liệu, định nghĩa target, ranh giới preprocessing và metric chính. So sánh với baseline tầm thường (majority class, mean hoặc giá trị trước đó) và linear model đơn giản. Ghi chi phí training/inference, calibration, error slice và độ bất định — không chỉ score cao nhất.

## 7. Chọn algorithm

- Văn bản thưa, nhiều chiều: linear model hoặc naive Bayes baseline.
- Dữ liệu dạng bảng cỡ trung: boosted tree và regularized linear model là ứng viên mạnh.
- Hình học trơn hoặc input image/text: có thể phù hợp với neural network.
- Cần quy tắc minh bạch: shallow tree hoặc linear model có constraint, kèm validation cẩn thận.
- Cần inference CPU độ trễ thấp: benchmark model gọn dưới workload thật.

Đây là giả thuyết khởi đầu, không phải luật. Chất lượng dữ liệu và thiết kế đánh giá thường quan trọng hơn họ thuật toán.

## 8. Bài tập

1. Vì sao decision tree sâu có thể đạt score train rất tốt nhưng test kém?
2. k-NN dùng tuổi theo năm và thu nhập theo đô-la. Cần cân nhắc gì trước khi tính distance?
3. Vì sao majority-class baseline hữu ích với classifier mất cân bằng?
4. Boosted model thắng chung nhưng thua nhiều ở group hiếm. Báo cáo chọn model cần bao gồm điều gì?
