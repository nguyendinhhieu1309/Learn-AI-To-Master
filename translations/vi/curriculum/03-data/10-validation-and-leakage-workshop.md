# Workshop có hướng dẫn về data validation và leakage

[Xem bài gốc bằng tiếng Anh](../../../../curriculum/03-data/10-validation-and-leakage-workshop.md)

Workshop này tập trung vào câu hỏi khó: **thông tin nào thực sự có sẵn vào lúc cần đưa ra dự đoán?** Chia tập đúng và tạo feature point-in-time quan trọng hơn model phức tạp.

## Mục tiêu học tập

- Chọn cách chia dữ liệu phù hợp với deployment.
- Phân biệt event time, thời điểm feature sẵn có, prediction time và label time.
- Phát hiện các dạng target leakage và group leakage thường gặp.
- Giải thích vì sao preprocessing chỉ được fit trên training data.

## 1. Xác định thời điểm dự đoán

Giả sử service dự đoán một đơn hàng có giao trễ hay không. Mỗi hàng đại diện cho một order ngay lúc nó được đặt, thời điểm $t_0$.

| Field | Ý nghĩa tại thời điểm dự đoán |
| --- | --- |
| order_id | ID đơn hàng duy nhất |
| customer_id | customer đã đặt hàng |
| order_time | timestamp của đơn hàng |
| promised_days | cam kết giao hàng đã biết lúc đặt |
| warehouse_id | kho đã gán nếu việc gán đã hoàn tất |
| delivered_time | chỉ quan sát được sau khi giao |
| late_label | được suy ra sau khi giao |

delivered_time không phải feature hợp lệ tại $t_0$. Dataset có thể sạch hoàn hảo nhưng vẫn không hợp lệ cho dự đoán nếu có thông tin được tạo sau thời điểm dự đoán.

Trước khi tạo feature, hãy viết contract dự đoán trong một câu:

> Với mỗi order, tại thời điểm đặt hàng, dự đoán đơn đó có giao trễ hơn ngày đã hứa không, chỉ dùng thông tin sẵn có vào thời điểm đó.

## 2. Bốn mốc thời gian không được nhầm lẫn

- **Event time:** lúc sự việc xảy ra trong thế giới thực.
- **Availability time:** lúc hệ thống lần đầu có thể sử dụng record.
- **Prediction time:** lúc model cần tạo score.
- **Label time:** lúc outcome trở nên xác định.

Ví dụ, payment event có thể xảy ra lúc 10:00 nhưng đến warehouse lúc 10:20. Prediction lúc 10:10 không thể dùng event đó, dù event timestamp sớm hơn prediction time.

Với mọi feature $x_j$, yêu cầu:

$$
\operatorname{available\_at}(x_j)\leq t_{\mathrm{prediction}}.
$$

Với label được tạo từ cửa sổ tương lai $[t_0,t_0+H]$, không đưa vào hàng có label window chưa trưởng thành tại cutoff dataset.

## 3. Chọn split theo quá trình tạo dữ liệu

### Ví dụ độc lập, có thể hoán đổi

Random split có thể hợp lý nếu ví dụ độc lập và dữ liệu tương lai được lấy từ cùng quá trình. Stratification có thể giữ tỷ lệ class gần đúng; nó không sửa được phụ thuộc hay time leakage.

### Người, thiết bị hoặc địa điểm xuất hiện nhiều lần

Nếu cùng customer xuất hiện trong cả train và test, model có thể ghi nhớ pattern riêng của customer. Dùng tách group khi deployment cần tổng quát hóa sang customer chưa gặp.

### Dự đoán tương lai

Nếu model dự đoán quan sát tương lai, train trên giai đoạn sớm và test trên giai đoạn muộn. Giữ một khoảng đệm thời gian nếu label hoặc feature chồng lấn qua ranh giới.

### Split là một phần của kết luận

“Model đạt accuracy 92%” là kết luận chưa đủ nếu thiếu đơn vị lấy mẫu và quy tắc split. Random row split, split theo customer mới và split theo thời gian tương lai trả lời những câu hỏi khác nhau.

## 4. Học preprocessing bên trong từng training fold

Giả sử bạn impute giá trị thiếu bằng mean của cột. Nếu tính mean bằng cả train lẫn validation, thông tin từ validation ảnh hưởng transformation đã học. Tác động có thể nhỏ, nhưng quy trình không còn mô phỏng một validation set thực sự chưa thấy.

Trình tự đúng:

1. Xác định train/validation/test split.
2. Fit imputation, scaling, feature selection và encoding trên train partition.
3. Áp dụng transformation đã fit cho validation hoặc test.
4. Chọn cấu hình bằng validation.
5. Sau khi chốt quyết định, fit lại pipeline đã chọn trên train cộng validation nếu protocol cuối yêu cầu.
6. Đánh giá test một lần.

Mọi lựa chọn dựa trên validation score — kể cả feature engineering và chọn threshold — đều là model selection, cần hoàn tất trước lần đánh giá test cuối.

## 5. Tìm leakage bằng cách hỏi giá trị được tạo ra thế nào

Với mỗi feature tiềm năng, trả lời:

1. Ai hoặc thứ gì đã tạo nó?
2. Nó trở nên khả dụng lúc nào?
3. Production có tính được nó cho mọi request không?
4. Nó có sao chép trực tiếp/gián tiếp target không?
5. Nó có chứa identifier hoặc trạng thái workflow sau outcome không?

Các pattern leakage phổ biến:

- trường status chỉ được cập nhật sau khi xử lý xong case;
- feature tính trên toàn dataset trước khi split;
- duplicate hoặc gần duplicate đi qua nhiều partition;
- target encoding dùng label held-out;
- event tương lai được đưa vào rolling aggregate;
- không tôn trọng label maturity tại cutoff;
- cùng patient, device, household hoặc robot session xuất hiện trong cả train lẫn test.

## 6. Ví dụ feature trượt có hướng dẫn

Với customer $c$ và thời điểm đặt order $t$, xét feature “số lần giao trễ trong 30 ngày trước đó”. Feature đúng chỉ dùng order sớm hơn mà outcome đã được biết:

$$
x(c,t)=\sum_{i:\;c_i=c,\;t-30d\leq t_i<t,\;\ell_i\text{ known by }t}\ell_i.
$$

Các điều kiện đều quan trọng:

- $t_i<t$ ngăn việc tính order hiện tại hoặc order tương lai.
- Cửa sổ 30 ngày giới hạn lịch sử.
- “Đã biết trước $t$” ngăn dùng outcome giao hàng chưa xảy ra hoặc chưa được hệ thống nhận.

Group-by ngây thơ trên mọi hàng có thể vi phạm cả ba điều kiện.

## 7. Kiểm tra chất lượng trước khi training

Ghi ít nhất:

- số hàng, tính duy nhất của key và grain của mỗi hàng;
- số lượng null/giá trị không hợp lệ theo cột;
- target prevalence theo thời gian và group;
- số duplicate và gần duplicate;
- timestamp nhỏ nhất/lớn nhất;
- quy tắc label maturity và feature availability;
- kích thước split và ID giao nhau giữa partition.

Không tự động xóa outlier. Trước tiên phân biệt giá trị bất khả thi, event hiếm nhưng hợp lệ và artifact đo lường.

## Bài tập

1. Fraud model dự đoán tại thời điểm transaction được cấp phép. Table có chargeback status, merchant category và chi tiêu tháng kế tiếp. Có thể dùng trường nào?
2. Mỗi patient có 10 ảnh được gán nhãn; mục tiêu là hiệu năng trên patient mới. Khi chia tập phải group theo gì?
3. Standardizer được fit trên toàn bộ dữ liệu trước cross-validation. Vì sao sai? Điểm số có nhất thiết tăng nhiều không?
4. Bạn dự đoán nhu cầu tuần sau với feature “doanh số trung bình mỗi cửa hàng”. Cần quy tắc timestamp và split nào?
5. Event xảy ra trước score nhưng được ingest sau score. Prediction có dùng được không?

## Kiểm tra đáp án

1. Merchant category dùng được nếu đã có tại thời điểm cấp phép. Chargeback status và chi tiêu tháng sau là thông tin hậu outcome, không hợp lệ. Kiểm tra thời điểm trường được điền, không chỉ dựa vào tên.
2. Group theo patient để tất cả ảnh của một người ở cùng partition.
3. Thống kê validation ảnh hưởng transformation đã fit. Đây là leakage trong quy trình đánh giá dù tác động lên score cuối có thể nhỏ.
4. Dùng split theo thời gian. Mean chỉ tính từ doanh số có sẵn trước cutoff của từng prediction; định nghĩa rõ đó là mean lịch sử, rolling hay aggregate theo giai đoạn train.
5. Không. Availability time, chứ không chỉ event time, quyết định hệ thống dự đoán có thể sử dụng dữ liệu.

## Mini-project

Chọn một dataset dạng bảng và tạo **data and split contract** một trang gồm:

1. đơn vị dự đoán và prediction time;
2. định nghĩa target và label-maturity window;
3. bảng feature availability;
4. quy tắc split và lý do;
5. kiểm tra duplicate/group overlap;
6. bước preprocessing chỉ fit trên training data;
7. nguồn bias hoặc missingness đã biết.

Đừng bắt đầu so sánh model cho đến khi một người học khác có thể review và phản biện contract này.

## Lỗi thường gặp

- Xem random split là lựa chọn mặc định cho mọi bài.
- So sánh record của cùng entity giữa partition mà không nêu mục tiêu deployment.
- Chỉ xét event time, bỏ qua độ trễ ingest.
- Gọi một biến là “metadata” rồi cho rằng nó không thể leak.
- Dùng lại test set để chọn feature, threshold hoặc model.
