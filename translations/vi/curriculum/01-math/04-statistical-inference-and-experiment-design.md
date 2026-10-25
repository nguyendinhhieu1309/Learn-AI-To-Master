# Suy luận thống kê và thiết kế thí nghiệm

> Bản tiếng Anh: [Statistical Inference and Experiment Design](../../../../curriculum/01-math/04-statistical-inference-and-experiment-design.md)

Thống kê giúp phân biệt một quy luật thật với biến động do lấy mẫu. Trong AI, ta dùng thống kê để so sánh mô hình, ước lượng độ bất định, thiết kế thu thập dữ liệu và quyết định một thay đổi có an toàn để phát hành hay không.

## 1. Tổng thể, mẫu và đại lượng cần ước lượng

- **Population (tổng thể):** tập các trường hợp mà ta muốn đưa ra kết luận, chẳng hạn mọi ticket hỗ trợ trong tương lai.
- **Sample (mẫu):** các trường hợp thực tế đã quan sát.
- **Parameter (tham số tổng thể):** thuộc tính của tổng thể, thường chưa biết.
- **Statistic (thống kê mẫu):** đại lượng tính được từ mẫu.
- **Estimand (đại lượng đích):** đại lượng cụ thể mà thí nghiệm muốn ước lượng, ví dụ mức thay đổi tỷ lệ xử lý thành công do mô hình mới gây ra.

Trước khi thu thập dữ liệu, hãy xác định đơn vị phân tích, tổng thể mục tiêu, outcome, phép so sánh và khoảng thời gian. Một phép kiểm định có mỗi hàng là một tin nhắn trả lời câu hỏi khác với phép kiểm định có mỗi hàng là một khách hàng.

Với các quan sát độc lập, sample mean $\\bar{x}$ và sample standard deviation $s$, standard error của mean là:

$$
SE(\\bar{x}) = \\frac{s}{\\sqrt{n}}
$$

Standard error mô tả độ bất định của ước lượng; nó không phải độ phân tán của từng quan sát. Nếu kích thước mẫu tăng gấp bốn lần mà cách lấy mẫu không đổi, standard error giảm còn một nửa.

## 2. Confidence interval

Quy trình confidence interval 95% có coverage 95% trong dài hạn nếu các giả định đúng. Điều đó **không** có nghĩa sau khi quan sát một khoảng cụ thể, tham số cố định có xác suất 95% nằm bên trong khoảng ấy.

Với mẫu đủ lớn, khoảng cơ bản cho mean là:

$$
\\bar{x} \\pm 1.96\\,SE(\\bar{x})
$$

Với mẫu nhỏ và dữ liệu gần chuẩn, dùng critical value của phân phối $t$ thay cho 1.96. Với tỷ lệ, dùng Wilson interval hoặc khoảng nhị thức phù hợp khác thay vì xấp xỉ chuẩn đơn giản khi số đếm ít hoặc tỷ lệ gần 0/1.

**Ví dụ.** Classifier phân loại đúng 820 trong 1.000 trường hợp độc lập. Accuracy quan sát được là $0.82$. Standard error gần đúng là $\\sqrt{0.82(0.18)/1000}\\approx0.012$. Khoảng 95% gần đúng là $0.82\\pm0.024$, tức 79,6%–84,4%. Độ bất định này đủ lớn để mức tăng một điểm phần trăm chưa thể tự nó chứng minh cải thiện đáng tin cậy.

## 3. Kiểm định giả thuyết và ý nghĩa thực tế

Đặt câu hỏi trước khi xem kết quả:

- Giả thuyết không $H_0$: không có tác động hoặc tác động bằng một mức baseline đã nêu.
- Giả thuyết đối $H_1$: có tác động theo hướng hoặc phạm vi quan tâm.
- Test statistic: đại lượng đã chuẩn hóa, đo kết quả quan sát khác kỳ vọng dưới $H_0$ ra sao.
- p-value: xác suất, giả sử $H_0$ và mô hình giả định là đúng, quan sát được dữ liệu ít nhất cũng không tương thích với $H_0$ như kết quả hiện tại.

p-value nhỏ không phải xác suất $H_0$ đúng và không đo độ lớn hay giá trị thực tế của tác động. Luôn báo cáo ước lượng tác động và khoảng tin cậy, rồi xét thay đổi đó có hữu ích khi tính latency, compute và chi phí lỗi hay không.

Với hai tỷ lệ độc lập $p_1,p_2$, standard error gần đúng cho hiệu số:

$$
SE(p_1-p_2) \\approx \\sqrt{\\frac{p_1(1-p_1)}{n_1} + \\frac{p_2(1-p_2)}{n_2}}
$$

Công thức mẫu độc lập không phù hợp với dữ liệu ghép cặp. Nếu hai hệ thống được chấm trên cùng trường hợp, hãy phân tích các hiệu số theo cặp hoặc dùng paired bootstrap.

## 4. Bootstrap và so sánh ghép cặp

Bootstrap xấp xỉ phân phối lấy mẫu bằng cách lấy lại các hàng **có hoàn lại** nhiều lần từ mẫu quan sát và tính lại statistic. Khi so sánh mô hình, lấy cùng các ví dụ cho cả hai mô hình để giữ cặp. Cách này ước lượng độ bất định của hiệu số chứ không coi hai điểm số là độc lập.

    observed_difference = metric(model_B, examples) - metric(model_A, examples)
    for b in 1..B:
        sample = resample_rows_with_replacement(examples)
        diff[b] = metric(model_B, sample) - metric(model_A, sample)
    interval = percentile(diff, [2.5, 97.5])

Bootstrap không sửa được dataset thiên lệch hoặc thiếu tính đại diện. Hãy resample đúng đơn vị độc lập: nếu một user có nhiều record, resample user chứ không resample từng hàng.

## 5. So sánh nhiều lần và xem kết quả lặp lại

Thử nhiều mô hình, threshold, prompt hoặc slice rồi chỉ báo cáo lựa chọn tốt nhất làm tăng xác suất phát hiện nhầm. Giữ riêng một final holdout set; dùng validation hoặc nested cross-validation để tuning. Trong thí nghiệm chính thức, xác định trước metric chính và dùng hiệu chỉnh hoặc kế hoạch kiểm định phân cấp khi có nhiều giả thuyết.

Liên tục kiểm tra p-value trên dữ liệu có cỡ mẫu cố định rồi dừng ngay khi qua ngưỡng cũng làm thay đổi false-positive rate. Hãy dùng quy trình sequential testing hoặc xác định cỡ mẫu và quy tắc dừng trước.

## 6. A/B test cho hệ thống AI

Randomize đơn vị không gây ảnh hưởng lẫn nhau. Ví dụ, gán cố định một khách hàng vào một hệ thống nếu các cuộc trò chuyện của cùng khách hàng ảnh hưởng qua lại. Theo dõi guardrail bên cạnh outcome chính: tỷ lệ lỗi, chuyển cho người xử lý, latency, cost và kết quả theo subgroup.

Chú ý:

- **Sample-ratio mismatch:** tỷ lệ phân bổ khác kế hoạch, có thể báo hiệu lỗi instrumentation hoặc gán nhóm.
- **Hiệu ứng mới lạ và thời gian:** hành vi người dùng hoặc lưu lượng thay đổi sau khi ra mắt.
- **Interference:** xử lý của một người làm ảnh hưởng outcome người khác.
- **Thiếu log:** mô hình có thể trông tốt hơn vì trường hợp khó không được ghi nhận.
- **Selection bias:** chỉ đánh giá trường hợp mô hình trả lời tự tin sẽ che khuất abstention và lỗi.

## 7. Bài tập

1. Mô hình tăng từ 91,0% lên 91,5% accuracy trên 400 ví dụ. Vì sao chưa đủ để khẳng định mức tăng đáng tin?
2. Hai mô hình chấm cùng 2.000 record. Vì sao paired bootstrap có thể nhiều thông tin hơn coi hai điểm số độc lập?
3. Nhóm thử 80 prompt trên test set rồi chọn prompt tốt nhất. Test set đã thay đổi vai trò ra sao, và cần thiết kế đánh giá tiếp theo thế nào?
4. A/B test randomize tin nhắn nhưng các outcome từ cùng user có tương quan. Nên randomize hoặc resample ở cấp nào?

## 8. Tự kiểm tra

1. Chỉ có point estimate và cỡ mẫu vẫn chưa đủ; cần mô hình lấy mẫu hoặc cách resample để ước lượng độ bất định. Mức tăng 0,5 điểm có thể chỉ là nhiễu lấy mẫu.
2. Pairing loại bớt biến động chung giữa trường hợp dễ/khó và ước lượng trực tiếp hiệu số từng trường hợp.
3. Test set đã tham gia chọn mô hình nên không còn là ước lượng cuối không thiên lệch. Thu một holdout mới hoặc dùng tập đánh giá ngoài được khóa.
4. Randomize và phân tích ở cấp user, hoặc dùng thiết kế cluster-aware để tính đến tương quan trong cùng user.
