# Làm sạch dữ liệu, EDA và feature engineering

[Xem bài gốc bằng tiếng Anh](../../../../curriculum/03-data/04-data-cleaning-eda-and-feature-engineering.md)

Model tốt bắt đầu từ bảng dữ liệu đáng tin cậy. Exploratory Data Analysis (EDA) là cách làm có kỷ luật để hiểu dữ liệu, phát hiện lỗi và hình thành giả thuyết mà không để test information rò vào training.

## 1. Xác định đơn vị quan sát và schema

Trước khi làm sạch, nêu rõ mỗi hàng đại diện cho điều gì, từng cột nghĩa là gì, key nào được dùng, dự đoán được đưa ra lúc nào và label được gán ra sao. Kiểm tra số hàng, key trùng, dtype, đơn vị, timezone và cách viết category. ID dạng số không nhất thiết là phép đo số học.

## 2. Giá trị thiếu

Missingness có thể là MCAR (không liên quan giá trị quan sát được hay không quan sát được), MAR (được giải thích bởi biến quan sát được) hoặc MNAR (liên quan chính giá trị bị thiếu). Cơ chế này ảnh hưởng tới modeling lẫn diễn giải.

Kiểm tra missingness theo label, thời gian, nguồn và cohort. Các lựa chọn gồm imputation, missing indicator tường minh, xử lý missing tích hợp trong model hoặc sửa quy trình thu thập. Chỉ fit giá trị imputation trên train data rồi áp dụng cùng transformation đã fit cho validation và production.

## 3. Duplicate, giá trị không hợp lệ và outlier

Khử trùng lặp theo key thực thể/sự kiện thực tế, không xóa mọi hàng có feature lặp. Phép đo lặp có thể hợp lệ. Kiểm tra constraint như số lượng không âm, ngày tháng hợp lý và các trường tương thích lẫn nhau.

Outlier có thể là lỗi, trường hợp hiếm nhưng hợp lệ hoặc trường hợp quan trọng nhất. Vẽ phân phối và kiểm tra ví dụ trước khi clipping hay xóa. Nếu dùng clipping hoặc robust scaling, hãy ước lượng ngưỡng trên train data và ghi rõ chính sách.

## 4. EDA không làm lộ test data

Dùng train data để khám phá feature và thiết kế transformation. Validation data có thể giúp chọn lựa, nhưng tuning quá nhiều có thể làm overfit validation; giữ test set cho đánh giá cuối. So sánh phân phối target và feature giữa các split để tìm leakage ngẫu nhiên hoặc sai khác population.

Các góc nhìn hữu ích gồm histogram, quantile, bản đồ missingness, pair plot cho số ít feature, class count, xu hướng thời gian và target rate theo category. Mỗi biểu đồ nên trả lời một câu hỏi. Mẫu tổng hợp có thể che lỗi ở một nhóm con, vì vậy cần kiểm tra các lát cắt có ý nghĩa.

## 5. Feature categorical và numerical

- One-hot encoding là baseline tốt cho category nominal có ít giá trị.
- Ordinal encoding chỉ hợp lệ khi thứ tự có ý nghĩa và được hiểu rõ.
- Frequency encoding hoặc target encoding cần cross-fitting cẩn thận để tránh label leakage.
- Log transform có thể hữu ích với feature dương, lệch phải.
- Standardization phổ biến với model dựa trên khoảng cách và linear model có regularization; tree split thường không cần.
- Biến chu kỳ như giờ trong ngày có thể mã hóa bằng sine/cosine nếu cần tính liên tục qua điểm quay vòng.

Với chu kỳ có period $T$, mã hóa giá trị $x$ thành:

$$
\left(\sin\frac{2\pi x}{T},\;\cos\frac{2\pi x}{T}\right)
$$

Cách này đặt giá trị đầu và cuối chu kỳ cạnh nhau về mặt hình học.

## 6. Tránh leakage từ target và thời gian

Hỏi xem mỗi feature đã tồn tại ở thời điểm quyết định chưa. Aggregate chỉ được dùng sự kiện đã xảy ra. Chẳng hạn “tổng lượt mua tháng này” gây leakage nếu dự đoán giữa tháng mà không tính riêng đến thời điểm đó. Dùng point-in-time join cho historical feature.

## 7. Xây preprocessing pipeline có thể tái lập

Chỉ fit transform đã học trên training fold rồi đóng gói chúng cùng model. Giữ tên và thứ tự feature ổn định. Thêm kiểm tra cột thiếu/không mong đợi, đơn vị, miền giá trị hợp lệ và output shape. Không lặp lại preprocessing thủ công trong notebook và production service.

## 8. Bài tập

1. Một feature thường thiếu hơn ở người dùng sẽ churn. Vì sao chỉ median imputation có thể che mất thông tin hữu ích?
2. Database có nhiều record mỗi customer và target là churn trong tương lai. Nên chia dữ liệu theo đơn vị nào?
3. Vì sao target encoding tính một lần trên toàn bộ training table trước cross-validation có thể gây leakage?
4. Biến thứ trong tuần được mã hóa integer 0 đến 6. Vì sao sine/cosine có thể hữu ích?
