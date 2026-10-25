# SQL và xây dựng dataset cho AI

[Xem bài gốc bằng tiếng Anh](../../../../curriculum/03-data/05-sql-and-dataset-construction.md)

SQL là thành phần trọng tâm khi xây training dataset từ hệ thống vận hành. Join đúng, lọc thời gian và cửa sổ gán nhãn quan trọng không kém thuật toán model.

## 1. Tư duy dữ liệu quan hệ

Table gồm hàng và cột; key định danh một thực thể hoặc sự kiện. Fact table ghi nhận sự kiện; dimension table bổ sung thuộc tính mô tả. Trước khi join, kiểm tra tính duy nhất của key và cardinality dự kiến. Join one-to-many có thể nhân hàng, vô tình tăng trọng số cho một số ví dụ.

## 2. Mẫu truy vấn cốt lõi

~~~sql
SELECT
    customer_id,
    COUNT(*) AS order_count,
    SUM(amount) AS total_amount,
    MAX(event_time) AS last_order_time
FROM orders
WHERE event_time < :prediction_time
GROUP BY customer_id;
~~~

Chỉ lọc thông tin có sẵn tại timestamp dự đoán. Với ví dụ training tại thời điểm $t$, từng feature phải tính được từ event có event time không muộn hơn $t$ và đã sẵn có cho hệ thống trước cutoff được định nghĩa.

Dùng LEFT JOIN nếu cần giữ lại ví dụ không có record liên quan; chỉ dùng INNER JOIN khi chủ ý loại hàng không khớp. Kiểm tra số hàng trước và sau từng join.

## 3. Window function

Window function tính aggregate cho từng hàng mà không gộp cả group thành một hàng. Chúng hữu ích để đếm event trước đó, xếp hạng và tính thống kê trượt. Chọn frame loại sự kiện hiện tại hoặc tương lai nếu dự đoán cần diễn ra trước event đó.

~~~sql
SELECT
    customer_id,
    event_time,
    SUM(amount) OVER (
        PARTITION BY customer_id
        ORDER BY event_time
        ROWS BETWEEN UNBOUNDED PRECEDING AND 1 PRECEDING
    ) AS amount_before_event
FROM orders;
~~~

Hành vi ở biên rất quan trọng: event đầu tiên không có event trước đó và cần được xử lý có chủ đích.

## 4. Label và cửa sổ quan sát

Định nghĩa feature window, prediction time, outcome window và độ trễ đến khi label trưởng thành. Ví dụ: dự đoán subscription có hủy trong 30 ngày không dựa trên activity của 90 ngày trước. Ví dụ gần đây có thể chưa đủ 30 ngày để outcome xảy ra; chúng bị kiểm duyệt hoặc chưa trưởng thành, không phải nhãn âm.

Tránh chọn hàng theo tình trạng dữ liệu tương lai theo cách gây thiên lệch population. Ghi lại các hàng bị loại và version label.

## 5. Trích xuất hiệu quả và an toàn

Chỉ chọn cột cần dùng; filter và aggregate gần nguồn dữ liệu khi phù hợp. Chia dataset lớn theo thời gian hoặc identifier ổn định. Parameterize query thay vì chèn chuỗi do người dùng cung cấp. Dùng transaction hoặc snapshot-consistent read nếu nhiều table cần phản ánh cùng một thời điểm.

## 6. Validation dataset đã dựng

Kiểm tra số hàng, sample key duy nhất, label prevalence, khoảng thời gian, missingness, target window, độ phủ join và cột có nguy cơ leakage. Lấy mẫu record rồi lần ngược thủ công về source event. Lưu version query/code, source snapshot và split assignment cùng dataset artifact.

## 7. Bài tập

1. Một customer có 10 transaction, một customer khác có 1. Join tạo 10 hàng cho customer đầu. Điều gì xảy ra nếu target ở mức customer?
2. Label churn 30 ngày được tạo cho event từ hôm qua. Vì sao gán “chưa churn” là không an toàn?
3. Vì sao feature theo window phải loại event sau timestamp dự đoán?
4. Kiểm tra nào phát hiện join bất ngờ làm số hàng dataset tăng gấp đôi?

### Gợi ý đáp án

1. Customer có nhiều transaction có thể bị tính như nhiều ví dụ hơn và bị overweight. Chọn rõ grain của label, rồi aggregate ở grain đó trước khi join.
2. Cửa sổ 30 ngày chưa trưởng thành; outcome chưa biết không đồng nghĩa với negative.
3. Vì sự kiện tương lai chưa có sẵn cho prediction, nên dùng nó gây leakage.
4. So sánh grain, tính duy nhất của key và số hàng trước/sau join; kiểm tra các key một-nhiều và tổng aggregate.
