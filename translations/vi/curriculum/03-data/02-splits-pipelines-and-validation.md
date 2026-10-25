# Chia tập dữ liệu, pipeline và validation

[Xem bài gốc bằng tiếng Anh](../../../../curriculum/03-data/02-splits-pipelines-and-validation.md)

## Mục tiêu học tập

Tạo cách chia tập phản ánh đúng việc sử dụng trong tương lai, giữ preprocessing đã học bên trong ranh giới train và biểu diễn giả định chất lượng dữ liệu thành các kiểm tra.

## 1. Vì sao cần chia dữ liệu?

Train data dùng để fit tham số model. Validation data giúp chọn model và setting. Test set cuối ước lượng hiệu năng sau khi đã hoàn tất quyết định. Dùng lại test set cho nhiều lựa chọn sẽ dần biến nó thành thông tin training.

Chỉ dùng random split khi các ví dụ độc lập và deployment giống việc lấy mẫu ngẫu nhiên. Dùng **group split** khi nhiều hàng thuộc cùng người/thiết bị/địa điểm và các group không được tách sang nhiều tập. Dùng **time split** khi dự đoán tương lai từ quá khứ. Stratification giúp giữ tỷ lệ class nếu phù hợp, nhưng không được lấn át ràng buộc quan trọng hơn về thời gian hoặc group.

## 2. Data leakage

Leakage xảy ra khi training dùng thông tin không có vào thời điểm dự đoán thực tế, hoặc thông tin từ ví dụ held-out. Ví dụ:

- scale toàn bộ hàng trước khi split;
- đưa sự kiện tương lai vào feature để dự đoán quá khứ;
- chia cùng người có nhiều record sang cả train và test;
- chọn feature sau khi xem hiệu năng trên test;
- vô tình để trường được tạo từ label trong input.

Hãy hỏi: “Đúng thời điểm cần đưa ra dự đoán này, trường dữ liệu đó đã tồn tại chưa?”

## 3. Chỉ fit transform trên train data

~~~python
from sklearn.compose import ColumnTransformer
from sklearn.impute import SimpleImputer
from sklearn.pipeline import make_pipeline
from sklearn.preprocessing import OneHotEncoder, StandardScaler
from sklearn.linear_model import LogisticRegression

numeric = ["age", "income"]
categorical = ["region"]

prep = ColumnTransformer([
    ("num", make_pipeline(SimpleImputer(strategy="median"), StandardScaler()), numeric),
    ("cat", make_pipeline(SimpleImputer(strategy="most_frequent"),
                           OneHotEncoder(handle_unknown="ignore")), categorical),
])
model = make_pipeline(prep, LogisticRegression(max_iter=1000))
model.fit(X_train, y_train)       # preprocessing learns only from training data
predictions = model.predict(X_test)
~~~

Pipeline gắn preprocessing với training model. Trong cross-validation, toàn bộ pipeline được fit lại riêng trong từng fold.

## 4. Validation theo constraint

Ghi rõ các quy tắc đơn giản phải luôn đúng: cột bắt buộc phải có, ID duy nhất nếu cần, ngày tháng parse được, miền số hợp lệ, tập category được biết (hoặc có cách xử lý unknown) và nhãn target được phép. Các kiểm tra này phát hiện thay đổi ở upstream trước khi kết quả âm thầm đổi khác.

Validation cần phân biệt **hard error** (cột target bắt buộc biến mất) và **warning** (tỷ lệ thiếu income tăng). Warning nên kích hoạt việc kiểm tra, không tự động xóa dữ liệu.

## 5. Khả năng tái lập và provenance

Ghi source URL/version, ngày truy cập nếu liên quan, license, filter, bước transformation, quy tắc split, random seed và schema đầu ra. Random seed giúp tái tạo split giả ngẫu nhiên, nhưng không đảm bảo kết quả giống hệt giữa phần cứng hay phiên bản thư viện khác nhau.

## Bài thực hành kèm gợi ý đáp án

Bạn dự đoán nhu cầu tháng sau từ record hằng ngày trong quá khứ. Random split có thể train bằng tháng sau rồi test trên tháng trước, khiến tác vụ dễ hơn forecasting thật. Nên chia theo thời gian: train trên ngày sớm hơn, validate trên cửa sổ tiếp theo và test trên cửa sổ mới nhất chưa đụng đến. Tính rolling statistic chỉ từ các hàng quá khứ.

## Lỗi thường gặp

- Dùng random split cho người dùng, thiết bị hoặc time series lặp lại mà không kiểm tra sự phụ thuộc.
- Fit scaling hoặc imputation trên toàn bộ dữ liệu trước cross-validation.
- Xóa ví dụ vi phạm giả định mà không báo số lượng hay lý do.
- Nhầm reproducibility với correctness: một data leak lặp lại được vẫn là leakage.
