# Unsupervised learning

[Xem bài gốc bằng tiếng Anh](../../../../curriculum/04-machine-learning/03-unsupervised-learning.md)

## Mục tiêu học tập

Hiểu clustering và dimensionality reduction có thể khám phá điều gì, cũng như vì sao cần diễn giải cẩn thận kết quả không có nhãn.

## 1. “Unsupervised” nghĩa là gì?

Phương pháp unsupervised tìm cấu trúc trong input mà không được cung cấp target label trong lúc fit. Chúng có thể gom nhóm ví dụ, tạo representation gọn hoặc gắn cờ điểm bất thường. Việc thuật toán trả về một pattern không tự chứng minh pattern đó có ý nghĩa.

## 2. Clustering

**K-means** gán điểm vào k center rồi lặp lại việc cập nhật assignment và center để giảm tổng bình phương khoảng cách trong cluster. Thuật toán đơn giản và hiệu quả nhưng giả định Euclidean distance có ý nghĩa, cần chọn k và ưu tiên cluster gần hình cầu. Khi đơn vị khác nhau, hãy scale feature số trước; nếu đang đánh giá pipeline, chỉ fit scaler trên training data.

Hierarchical clustering tạo hệ thống nhóm lồng nhau. Phương pháp dựa trên mật độ như DBSCAN có thể tìm cluster hình dạng bất quy tắc và gắn một số điểm là noise, nhưng phụ thuộc distance và density setting.

## 3. Dimensionality reduction

PCA tìm các hướng trực giao giữ variance cao; PCA có thể nén hoặc trực quan hóa dữ liệu. Variance cao không nhất thiết có tính dự đoán hoặc ý nghĩa ngữ nghĩa. Phương pháp trực quan hóa phi tuyến như t-SNE hay UMAP giúp thấy pattern lân cận nhưng có thể làm méo khoảng cách toàn cục. Xem biểu đồ 2D là công cụ khám phá, không phải bằng chứng dữ liệu có chính xác ba nhóm tự nhiên.

## 4. Phát hiện bất thường

Anomaly detection tìm điểm khác với tập tham chiếu đã học. Hiếm không đồng nghĩa sai hay nguy hiểm. Định nghĩa rõ anomaly có ý nghĩa gì khi vận hành và kiểm tra ví dụ. Nếu có anomaly label, đánh giá bằng precision-recall phù hợp; nếu không, mời chuyên gia domain xem và kiểm tra tính ổn định.

## 5. Đánh giá khi không có ground truth

Kiểm tra tính ổn định qua seed và thay đổi nhỏ trong dữ liệu, khả năng diễn giải cluster, ích lợi cho tác vụ sau và nhận xét của chuyên gia. Metric nội tại như silhouette score tóm tắt độ tách biệt hình học theo distance đã chọn; chúng không chứng minh ích lợi khoa học hay kinh doanh.

## Thực hành

Cluster một dataset nhỏ hai lần: một lần dùng feature thô, một lần dùng feature đã scale. So sánh assignment và giải thích feature nào chi phối distance. Sau đó xem một số ví dụ mỗi cluster và đề xuất tên mô tả, không khẳng định quan hệ nhân quả.

## Lỗi thường gặp

- Chọn số cluster chỉ vì biểu đồ nhìn đẹp.
- Dùng phương pháp dựa trên distance với feature chưa scale, đơn vị không tương thích.
- Diễn giải PCA component hoặc trục t-SNE như biến ban đầu.
- Coi anomaly score là label lỗi đã được xác minh.
