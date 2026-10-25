# Array và bảng dữ liệu với NumPy và pandas

[Xem bài gốc bằng tiếng Anh](../../../../curriculum/02-python/02-arrays-and-tables.md)

## Mục tiêu học tập

Hiểu sự khác nhau giữa Python list, NumPy array và bảng pandas; chọn dữ liệu theo vị trí hoặc tên; kiểm tra shape và giá trị thiếu.

## 1. NumPy array

NumPy array lưu dữ liệu số theo shape đều đặn và hỗ trợ phép toán vector hóa:

~~~python
import numpy as np

x = np.array([1.0, 2.0, 3.0])
weights = np.array([0.5, -1.0, 2.0])
score = x @ weights          # dot product: 4.5
scaled = 2 * x               # [2.0, 4.0, 6.0]
~~~

Shape của x là (3,). Với matrix, shape của X có thể là (n_examples, n_features). Vectorization giúp code gọn và thường nhanh hơn vòng lặp Python, nhưng tuân theo quy tắc broadcasting. Nếu shape kết quả gây bất ngờ, hãy in shape ra trước khi tiếp tục.

## 2. Bảng pandas

DataFrame là bảng có hàng và cột được gắn nhãn. Series là một cột có nhãn.

~~~python
import pandas as pd

df = pd.DataFrame({
    "hours": [2.0, 4.0, 3.0],
    "passed": [False, True, True],
})
print(df.shape)          # (3, 2)
print(df["hours"].mean())
print(df.isna().sum())
~~~

Các kiểm tra ban đầu hữu ích gồm df.head(), df.shape, df.dtypes, df.describe(include="all"), df.isna().sum() và df.nunique(). Chúng giúp phát hiện cột đáng ngờ, dữ liệu thiếu và category bất ngờ trước khi modeling.

## 3. Chọn theo nhãn và điều kiện

Dùng df["column"] để chọn một cột. Dùng .loc[rows, columns] để chọn theo nhãn và .iloc[rows, columns] để chọn theo vị trí số nguyên. Lọc hàng bằng điều kiện Boolean:

~~~python
passed = df.loc[df["hours"] >= 3, ["hours", "passed"]]
~~~

Khi ghép điều kiện, đặt từng điều kiện trong ngoặc và dùng & cho AND, | cho OR. Không dùng and của Python với toàn bộ cột pandas.

## 4. Nạp dữ liệu và chú ý kiểu dữ liệu

~~~python
df = pd.read_csv("data/students.csv")
~~~

Kiểm tra các kiểu pandas tự suy luận. Cột số có thể được đọc thành text nếu chứa giá trị như "unknown" hoặc dấu phẩy. Hãy chuyển đổi có chủ đích bằng pd.to_numeric(..., errors="coerce"), sau đó kiểm tra giá trị nào đã trở thành missing thay vì âm thầm bỏ chúng.

## Bài tập ngắn

Tạo bảng gồm ba sản phẩm, giá và số lượng. Thêm cột revenue bằng price nhân quantity, lọc sản phẩm có doanh thu trên 100 và tính tổng doanh thu. Kiểm tra đơn vị có nhất quán không, chẳng hạn đô-la nhân số lượng sản phẩm.

## Lỗi thường gặp

- Đoán shape và ý nghĩa của axis mà không kiểm tra.
- Coi giá trị thiếu là 0 khi không có lý do nghiệp vụ.
- Dùng chained assignment trong pandas; hãy tạo bản sao tường minh hoặc dùng .loc để chỉ rõ cập nhật mong muốn.
- Tính toán trên cột chuỗi rồi tin kết quả mà chưa kiểm tra kiểu dữ liệu.
