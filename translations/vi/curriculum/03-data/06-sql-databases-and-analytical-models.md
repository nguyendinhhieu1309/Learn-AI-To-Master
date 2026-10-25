# SQL, database và mô hình dữ liệu phân tích

[Xem bài gốc bằng tiếng Anh](../../../../curriculum/03-data/06-sql-databases-and-analytical-models.md)

## Mục tiêu học tập

Truy vấn dữ liệu liên quan mà không vô tình nhân số hàng, hiểu workload giao dịch và phân tích, mô hình hóa dữ liệu để báo cáo và ML đáng tin cậy.

## 1. Bảng quan hệ và key

Table gồm hàng và cột. **Primary key** định danh duy nhất một hàng. **Foreign key** tham chiếu tới hàng ở table khác. Trước khi join, hãy hiểu grain của mỗi table: một hàng trên customer, order, order line hay event? Grain quyết định phép aggregate nào hợp lệ.

Ví dụ: customers(customer_id, region) và orders(order_id, customer_id, amount, created_at). customer_id có thể lặp trong orders vì một customer có thể đặt nhiều order.

## 2. Cardinality của join

Inner join giữ key khớp; left join giữ mọi hàng bên trái rồi bổ sung giá trị khớp bên phải. Nếu cả hai table có nhiều hàng cho cùng key join, một hàng có thể khớp nhiều hàng và nhân kết quả. Kiểm tra tính duy nhất và số hàng trước khi aggregate.

~~~sql
SELECT c.region, COUNT(DISTINCT o.order_id) AS order_count,
       SUM(o.amount) AS revenue
FROM customers AS c
JOIN orders AS o USING (customer_id)
WHERE o.created_at >= DATE '2025-01-01'
GROUP BY c.region
ORDER BY revenue DESC;
~~~

Với dữ liệu order line, hãy cộng amount ở grain của line hoặc aggregate line thành một hàng mỗi order trước khi join thêm table one-to-many khác.

## 3. SQL thiết yếu

- SELECT chọn expression/cột; WHERE lọc hàng nguồn.
- GROUP BY tạo group aggregate; HAVING lọc group sau aggregation.
- ORDER BY sắp xếp kết quả; LIMIT giới hạn số hàng trả về ở nhiều hệ thống.
- NULL nghĩa là không biết hoặc thiếu; kiểm tra bằng IS NULL, không phải = NULL.
- Dùng tên cột và alias tường minh; tránh SELECT * trong contract pipeline ổn định lâu dài.

Dùng parameterized query cho giá trị do người dùng cung cấp. Không bao giờ tạo SQL bằng cách nối raw user text.

## 4. Hệ thống giao dịch và phân tích

OLTP xử lý nhiều thao tác đọc/ghi nhỏ cho transaction của ứng dụng. OLAP tối ưu việc quét và aggregate lịch sử lớn. Data warehouse thường tổ chức record đã làm sạch, tích hợp để phân tích. Star schema có fact table trung tâm (event/measure) và các dimension table (ai/cái gì/ở đâu/khi nào). Nhờ đó grain rõ ràng và giảm join mâu thuẫn.

Index có thể tăng tốc tra cứu hoặc join nhưng tốn bộ nhớ và làm ghi chậm. Định dạng columnar như Parquet đọc các cột được chọn hiệu quả; table theo hàng phù hợp với truy cập giao dịch. Chọn dựa trên query, pattern cập nhật và nhu cầu vận hành.

## 5. Mô hình dữ liệu cho ML

Tạo prediction table tường minh với một hàng cho mỗi thời điểm quyết định, timestamp/cutoff, feature có sẵn tại thời điểm đó và target được định nghĩa trong cửa sổ tương lai. Giữ entity ID và source timestamp đủ lâu để validation grouping/leakage, dù không dùng chúng làm feature của model. Ghi cách tạo label và thời gian trễ.

## Thực hành

Với hai table có key lặp, hãy dự đoán số hàng sau inner join trước khi chạy. Kiểm tra số lượng key, thực hiện join, so sánh số hàng và tổng amount trước/sau, rồi sửa query nếu amount bị nhân.

## Lỗi thường gặp

- Join bằng key không duy nhất làm nhân fact.
- Coi NULL, zero và “không áp dụng” là cùng một giá trị.
- Đưa dữ liệu sau outcome vào feature table.
- Tạo analytics table mà không tài liệu hóa grain.
