# Xác suất và thống kê

> Bản tiếng Anh: [Probability and Statistics](../../../../curriculum/01-math/03-probability-and-statistics.md)

## Mục tiêu học tập

Dùng xác suất để mô tả độ bất định, thống kê để tóm tắt dữ liệu quan sát và mẫu để suy luận thận trọng về một tổng thể lớn hơn.

## 1. Xác suất và biến cố

Xác suất nằm trong khoảng từ 0 (không thể xảy ra) đến 1 (chắc chắn). Với xúc xắc sáu mặt cân bằng, $P(X=4)=1/6$. Nếu hai biến cố không thể xảy ra cùng lúc thì $P(A\\cup B)=P(A)+P(B)$. Trong trường hợp tổng quát:

$$
P(A \\cup B)=P(A)+P(B)-P(A \\cap B).
$$

Phần giao được trừ đi để tránh đếm hai lần các kết quả thuộc cả hai biến cố.

## 2. Xác suất có điều kiện và Bayes

Xác suất có điều kiện $P(A\\mid B)$ là xác suất xảy ra biến cố A khi đã biết B. Định lý Bayes đảo chiều điều kiện:

$$
P(A\\mid B)=\frac{P(B\\mid A)P(A)}{P(B)}.
$$

Giả sử một bệnh hiếm gặp ở 1% dân số. Xét nghiệm có sensitivity 90% ($P(+\\mid C)=0.9$) và tỷ lệ dương tính giả 5% ($P(+\\mid\\neg C)=0.05$). Trong 10.000 người, kỳ vọng khoảng 90 dương tính thật và 495 dương tính giả. Xác suất mắc bệnh khi xét nghiệm dương tính là:

$$
P(C\\mid+)=\frac{90}{90+495}\approx0.154,
$$

khoảng 15,4%, không phải 90%. Base rate rất quan trọng.

## 3. Biến ngẫu nhiên, kỳ vọng và phương sai

Biến ngẫu nhiên biểu diễn một kết quả số chưa biết trước. Kỳ vọng là trung bình dài hạn. Với biến ngẫu nhiên rời rạc:

$$
\mathbb{E}[X]=\sum_x xP(X=x).
$$

Phương sai đo độ phân tán quanh mean; standard deviation là căn bậc hai của phương sai:

$$
\operatorname{Var}(X)=\mathbb{E}\left[(X-\mathbb{E}[X])^2\right],
\qquad \sigma_X=\sqrt{\operatorname{Var}(X)}.
$$

Hai dataset có thể cùng mean nhưng độ phân tán rất khác. Khi variability quan trọng, không nên chỉ báo cáo trung bình.

## 4. Mẫu và ước lượng

Dataset thường chỉ là một mẫu, không phải toàn bộ population mà ta quan tâm. Sample mean thay đổi giữa các mẫu. Nhiều quan sát độc lập và có tính đại diện thường làm ước lượng ổn định hơn; một mẫu lớn nhưng thiên lệch vẫn có thể gây hiểu lầm.

Confidence interval mô tả một quy trình mà nếu lặp lại trên nhiều mẫu, dưới các giả định của nó, sẽ bao phủ tham số thật với tỷ lệ đã nêu. Sau khi tính được một khoảng cụ thể, không nên diễn giải rằng tham số cố định có xác suất 95% nằm trong khoảng ấy.

Với các quan sát độc lập, standard error của sample mean ước tính mức biến động của mean giữa các mẫu lặp:

$$
SE(\bar{x})=\frac{s}{\sqrt{n}}.
$$

Với mẫu đủ lớn, khoảng gần đúng 95% cho mean là $\\bar{x}\\pm1.96SE(\\bar{x})$. Với mẫu nhỏ hoặc tỷ lệ gần 0 hay 1, hãy dùng phương pháp khoảng tin cậy phù hợp với giả định dữ liệu thay vì áp dụng máy móc xấp xỉ này.

## 5. Liên hệ không đồng nghĩa nhân quả

Correlation mô tả hai đại lượng biến thiên cùng nhau. Điều đó có thể do một đại lượng gây ra đại lượng kia, cả hai cùng chịu tác động của nguyên nhân khác, do selection effect hoặc tình cờ. Mô hình dự đoán có thể dùng feature tương quan với target mà không chứng minh tác động nhân quả.

## 6. Đánh giá ML và độ bất định

Dự đoán và metric luôn có độ bất định vì dữ liệu hữu hạn và dữ liệu triển khai có thể khác. Dùng test set giữ riêng để đưa ra ước lượng cuối; cân nhắc nhiều lần chia mẫu hoặc bootstrap interval nếu kích thước mẫu cho phép. Tránh chọn mô hình lặp đi lặp lại dựa vào test set vì khi đó test đã tham gia quá trình chọn mô hình.

## Tự kiểm tra

1. Bệnh xuất hiện ở 1/100 người. Vì sao kết quả dương tính từ xét nghiệm khá chính xác vẫn có thể sai? **Đáp án:** false positive trong nhóm không bệnh đông hơn nhiều có thể vượt số true positive.
2. Correlation có chứng minh feature gây ra target dự đoán không? **Đáp án:** không.
3. Nếu dữ liệu chỉ lấy từ một bệnh viện, điều gì có thể không đúng ở nơi khác? **Đáp án:** tính đại diện và các giả định về phân phối.

## Lỗi thường gặp

- Nhầm $P(A\\mid B)$ với $P(B\\mid A)$.
- Coi thống kê mẫu là giá trị chính xác của tổng thể.
- Chỉ báo cáo mean mà che giấu độ phân tán hoặc chênh lệch giữa nhóm.
- Diễn giải liên hệ dự đoán như một kết luận nhân quả.
