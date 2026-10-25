# Unsupervised learning và anomaly detection

[Xem bài gốc bằng tiếng Anh](../../../../curriculum/04-machine-learning/06-unsupervised-learning-and-anomaly-detection.md)

Unsupervised learning tìm cấu trúc mà không có target label. Kết quả thường mang tính mô tả hơn dự đoán; validation cần định nghĩa tác vụ cẩn thận và review theo domain.

## 1. Clustering

K-means tối thiểu hóa bình phương khoảng cách trong cluster tới centroid. Phương pháp hiệu quả nhưng cần chọn $k$, ưu tiên cluster gần hình cầu và nhạy với scale/khởi tạo. Chạy nhiều lần khởi tạo và kiểm tra độ ổn định cluster.

Hierarchical clustering tạo cây gộp hỗ trợ nhiều mức chi tiết. DBSCAN nhóm vùng mật độ cao và đánh dấu điểm thưa là noise; có thể tìm cluster không hình cầu nhưng nhạy với scale distance và mật độ thay đổi. Gaussian mixture model ước lượng xác suất mềm thuộc cluster theo giả định phân phối hỗn hợp.

## 2. Dimensionality reduction

PCA tạo phép chiếu tuyến tính giữ variance cao. Nó hữu ích cho nén, khử nhiễu và trực quan hóa, nhưng hướng variance cao không nhất thiết dự đoán tốt hay có ý nghĩa ngữ nghĩa.

t-SNE và UMAP hỗ trợ trực quan hóa neighborhood cục bộ, nhưng khoảng cách và kích thước cluster trên biểu đồ không nên được coi là hình học toàn cục chính xác. Dùng visualization để khám phá; đừng xem biểu đồ tách biệt rõ là bằng chứng duy nhất cho cluster tự nhiên.

## 3. Anomaly detection

Anomaly được định nghĩa tương đối với một quy trình và hậu quả. Hiếm không nhất thiết là sai hay nguy hiểm. Isolation Forest cô lập điểm bằng phép chia ngẫu nhiên; One-Class SVM học biên quanh ví dụ điển hình; phương pháp reconstruction gắn cờ input tái tạo kém. Mỗi cách có giả định và có thể thất bại khi dữ liệu bình thường bị shift hoặc đa mode.

Khi có anomaly label, đánh giá precision/recall tại threshold vận hành, số alert và thời gian phát hiện. Khi thiếu label, tổ chức expert review, test tổng hợp có kiểm soát và giám sát gánh nặng false alarm. Tránh chọn threshold chỉ từ training score.

## 4. Scaling và representation

Phương pháp dựa trên distance nhạy với đơn vị feature và chiều không liên quan. Standardize bằng thống kê train. Với dữ liệu nhiều chiều, chọn hoặc học representation cẩn thận và xác minh kết quả bền vững qua seed/sample.

## 5. Bài tập

1. K-means tách cluster theo thu nhập nhưng bỏ qua mã vùng dạng categorical. Cần cân nhắc gì về preprocessing và model?
2. UMAP cho thấy ba “hòn đảo”. Vì sao chưa đủ để kết luận có ba population tự nhiên?
3. Anomaly detector có precision 1%. Cần tính chi phí vận hành nào trước deployment?
4. Nhóm fraud không có label cho anomaly mới nhất. Có thể tạo vòng review nào để xây evaluation data an toàn?
