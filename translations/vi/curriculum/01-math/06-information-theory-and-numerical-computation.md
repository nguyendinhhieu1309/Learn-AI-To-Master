# Lý thuyết thông tin và tính toán số

Lý thuyết thông tin giải thích độ bất định và mã hóa; numerical analysis giải thích vì sao công thức tương đương trong Toán có thể hoạt động khác nhau trên máy tính.

## 1. Entropy và cross-entropy

Với biến ngẫu nhiên rời rạc có xác suất $p_i$, entropy là:

$$
H(P)=-\sum_i p_i\log p_i
$$

Entropy cao khi xác suất phân tán trên nhiều kết quả, thấp khi một kết quả chiếm ưu thế. Cơ số logarithm quyết định đơn vị: cơ số 2 cho bit, logarithm tự nhiên cho nat.

Cross-entropy giữa phân phối target P và phân phối dự đoán Q:

$$
H(P,Q)=-\sum_i p_i\log q_i
$$

Với label one-hot, tối thiểu hóa cross-entropy khuyến khích xác suất cao cho lớp đúng. KL divergence đo chi phí mã hóa tăng thêm khi dùng Q thay P:

$$
D_{\mathrm{KL}}(P\|Q)=\sum_i p_i\log\frac{p_i}{q_i}
=H(P,Q)-H(P)
$$

KL divergence không âm nhưng bất đối xứng, nên không phải khoảng cách thông thường.

## 2. Log-probability và softmax ổn định

Tính trực tiếp $e^{z_i}$ có thể overflow khi logit lớn. Trừ logit lớn nhất không đổi softmax nhưng cải thiện ổn định:

$$
\operatorname{softmax}(z_i)=
\frac{e^{z_i-m}}{\sum_j e^{z_j-m}},\qquad m=\max_j z_j
$$

Dùng log_softmax ổn định hoặc hàm cross-entropy nhận logits của framework thay vì tự tính log(softmax(...)). Log-sum-exp cũng giúp tính tổng ổn định.

## 3. Giới hạn của floating point

Máy tính lưu số floating-point xấp xỉ. Sai số làm tròn có thể tích lũy; cộng một số rất nhỏ vào số rất lớn có thể làm mất phần nhỏ. Các cách biến đổi đại số tương đương có thể gây sai số số học khác nhau. Tránh so sánh chính xác bằng nhau với số thực đã tính; dùng tolerance phù hợp scale và mục đích.

Vấn đề thường gặp khác gồm overflow, underflow, chia cho số gần 0, ma trận ill-conditioned và đầu vào không hữu hạn. Theo dõi loss và gradient có hữu hạn không. Chuẩn hóa scale đầu vào khi phù hợp và ưu tiên routine ổn định của thư viện thay vì tự viết phép nghịch đảo ma trận.

## 4. Vì sao không nên tính matrix inverse?

Để giải $Ax=b$, dùng linear solver thay vì tạo tường minh $A^{-1}$ rồi nhân. Solver thường nhanh và ổn định hơn, đồng thời có thể tận dụng cấu trúc ma trận. Nếu A suy biến hoặc condition number xấu, hãy xem lại mô hình/dữ liệu hoặc cân nhắc regularization và cách biểu diễn khác.

## 5. Entropy trong thực hành ML

Entropy xuất hiện trong phép chia của decision tree, ước lượng bất định, language modeling và policy learning có regularization. Predictive entropy một mình không cho biết mô hình sai hay đúng: mô hình tự tin nhưng sai vẫn có entropy thấp. Hãy so sánh phương pháp đo bất định với lỗi thực tế và calibration trên dữ liệu giữ riêng.

## Bài tập

1. Entropy của kết quả nhị phân chắc chắn bằng bao nhiêu? Với đồng xu cân bằng, entropy theo bit bằng bao nhiêu?
2. Vì sao classification thông thường dùng cross-entropy với xác suất lớp thay squared error?
3. Vì sao softmax([1000,1001]) có thể overflow khi tính trực tiếp, và trừ giá trị lớn nhất giúp gì?
4. Vì sao nên tránh tính inverse tường minh khi giải $Ax=b$?
