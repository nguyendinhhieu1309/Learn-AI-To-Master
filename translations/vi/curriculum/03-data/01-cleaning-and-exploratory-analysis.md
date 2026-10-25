# Làm sạch và khám phá một dataset dạng bảng

[Xem bài gốc bằng tiếng Anh](../../../../curriculum/03-data/01-cleaning-and-exploratory-analysis.md)

## Mục tiêu học tập

Bắt đầu từ bảng dữ liệu chưa quen thuộc, tìm vấn đề chất lượng, đưa ra quyết định làm sạch có cơ sở và tóm tắt điều dữ liệu có thể cho biết.

## 1. Xác định ý nghĩa của mỗi hàng

Trước khi vẽ biểu đồ, hãy ghi rõ đơn vị quan sát: mỗi hàng là một người, đơn hàng, lần đọc cảm biến hay ngày? Xác định target nếu có, khoảng thời gian, quy trình thu thập và mục đích sử dụng. Nếu mỗi người có nhiều hàng, việc chia ngẫu nhiên theo hàng về sau có thể đưa cùng một người vào cả train lẫn test.

## 2. Kiểm tra trước khi thay đổi

~~~python
import pandas as pd

df = pd.read_csv("data/example.csv")
print(df.shape)
print(df.head())
print(df.dtypes)
print(df.isna().sum().sort_values(ascending=False))
print(df.nunique(dropna=False))
~~~

Kiểm tra ID duy nhất, bản ghi trùng, đơn vị đo, miền giá trị bất khả thi, chính tả không nhất quán và việc parse ngày tháng. Giá trị ngoài khoảng thường gặp là câu hỏi cần điều tra, không tự động là lỗi.

## 3. Giá trị thiếu

Thiếu dữ liệu có thể nghĩa là “chưa đo”, “không áp dụng”, cảm biến lỗi hoặc giá trị được ẩn vì quyền riêng tư. Các ý nghĩa này khác nhau. Hãy đo tần suất thiếu và xem thiếu có tập trung ở nhóm hoặc giai đoạn nào không.

Các lựa chọn có thể gồm: giữ missing category; impute bằng median số hoặc giá trị categorical phổ biến nhất; thêm missingness indicator; hoặc loại hàng/cột nếu có cơ sở. Khi đánh giá dự đoán, chỉ học giá trị imputation từ train rồi áp dụng nguyên trạng cho validation/test. Nếu không, thông tin từ held-out data bị leak vào training.

## 4. Bản ghi trùng, kiểu dữ liệu và outlier

Các hàng trùng chính xác có thể do lỗi, nhưng hàng lặp cũng có thể là sự kiện hợp lệ. Trước khi xóa duplicate, kiểm tra key lẽ ra phải duy nhất. Parse ngày và category có chủ đích. Với outlier số, dùng biểu đồ và quy tắc domain; quan sát cực đoan nhưng có thật có thể chính là trường hợp model cần xử lý.

Ví dụ: nếu tài liệu cảm biến quy định miền 0–100 thì giá trị 999 có thể là mã lỗi. Nếu không có miền được tài liệu hóa, đừng clip chỉ vì giá trị trông lạ.

## 5. Khám phá phân phối và mối quan hệ

Dùng histogram hoặc quantile cho feature số, bar chart cho category và scatter plot cho từng cặp. So sánh target giữa các nhóm liên quan. Luôn xem số lượng mẫu: mean nhiễu từ ba ví dụ không nên được diễn giải như mean từ hàng nghìn ví dụ.

EDA gợi ra câu hỏi; nó không xác nhận giải thích nhân quả. Nếu feature liên quan mạnh đến target, nó có thể là proxy, artifact từ cách thu thập label hoặc biến chỉ xuất hiện sau outcome.

## Ví dụ làm sạch có hướng dẫn

~~~python
df["age"] = pd.to_numeric(df["age"], errors="coerce")
df["joined_at"] = pd.to_datetime(df["joined_at"], errors="coerce", utc=True)

invalid_age = df["age"].notna() & ~df["age"].between(0, 120)
print("Invalid ages:", int(invalid_age.sum()))
print("Missing age:", int(df["age"].isna().sum()))
~~~

Code này gắn cờ dữ liệu đáng nghi; không âm thầm xóa nó. Tiếp theo hãy kiểm tra nguồn, quyết định cách xử lý các hàng này rồi ghi lại quy tắc.

## Bài thực hành

Với dataset công khai bất kỳ, lập báo cáo kiểm tra dữ liệu một trang gồm: ý nghĩa hàng, target, kiểu cột, giá trị thiếu, kiểm tra key/trùng, giá trị đáng nghi, class balance nếu có và ba biểu đồ. Thêm một câu hỏi mà dataset không thể trả lời đáng tin cậy.

## Lỗi thường gặp

- Xóa mọi hàng thiếu ít nhất một giá trị mà chưa đo số lượng bị mất.
- Thay mọi giá trị lạ bằng median hoặc một ngưỡng biên.
- Vẽ mối quan hệ mà chưa kiểm tra cỡ mẫu hay thiên lệch thu thập.
- Làm sạch label theo hướng khiến accuracy của model trông tốt hơn.
