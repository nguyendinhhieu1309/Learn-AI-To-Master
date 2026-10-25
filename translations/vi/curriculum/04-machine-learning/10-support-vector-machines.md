# Support Vector Machine

[Xem bài gốc bằng tiếng Anh](../../../../curriculum/04-machine-learning/10-support-vector-machines.md)

Support Vector Machine (SVM) là model dựa trên margin. Hình học của nó giải thích objective, độ nhạy với scaling và cách kernel tạo decision boundary phi tuyến.

## Mục tiêu học tập

- Viết linear decision boundary và đo khoảng cách điểm tới biên.
- Giải thích objective maximum-margin và support vector.
- Phân biệt hard-margin với soft-margin SVM.
- Diễn giải hinge loss, đánh đổi regularization và kernel similarity.
- Biết khi nào SVM phù hợp và cách đánh giá.

## 1. Hyperplane phân tách

Với feature $x\in\mathbb{R}^{d}$, linear boundary là:

$$
w^Tx+b=0.
$$

Phía được dự đoán xác định theo dấu của $w^Tx+b$. Khoảng cách vuông góc từ điểm $x$ tới boundary:

$$
\frac{|w^Tx+b|}{\|w\|_2}.
$$

Với label $y_i\in\{-1,+1\}$, signed margin là $y_i(w^Tx_i+b)/\|w\|_2$. Giá trị dương nghĩa là ví dụ ở phía đúng; giá trị càng lớn, ví dụ càng xa boundary.

## 2. Maximum margin

Nếu dữ liệu phân tách tuyến tính hoàn hảo, scale $w,b$ sao cho điểm gần nhất thỏa $y_i(w^Tx_i+b)\geq1$. Tối đa hóa geometric margin tương đương tối thiểu hóa norm của weight:

$$
\min_{w,b}\frac12\|w\|_2^2
\quad\text{subject to}\quad
y_i(w^Tx_i+b)\geq1\quad\forall i.
$$

Hệ số $\frac12$ làm đạo hàm gọn. **Support vector** là điểm training nằm trên hoặc bên trong margin; chúng quyết định boundary được fit. Điểm cách margin xa thường không làm thay đổi nghiệm hard-margin chính xác.

## 3. Soft margin và hinge loss

Dataset thực thường chồng lấn và có noise. Thêm slack variable $\xi_i\geq0$ cho vi phạm margin:

$$
\min_{w,b,\xi}\frac12\|w\|_2^2+C\sum_i\xi_i
\quad\text{subject to}\quad
y_i(w^Tx_i+b)\geq1-\xi_i.
$$

Objective tương đương dùng hinge loss:

$$
\min_{w,b}\frac12\|w\|_2^2
+C\sum_i\max\left(0,1-y_i(w^Tx_i+b)\right).
$$

$C$ kiểm soát đánh đổi. $C$ lớn phạt vi phạm margin mạnh, có thể fit training chặt hơn; $C$ nhỏ regularize mạnh và cho phép nhiều vi phạm hơn. Thư viện có thể viết objective dạng tổng hoặc trung bình; đừng so raw $C$ giữa implementation mà chưa xem quy ước.

### Kiểm tra margin nhỏ

Với $y=+1$ và score $w^Tx+b=1.4$, hinge loss là $\max(0,1-1.4)=0$. Với ví dụ âm $y=-1$ có score 0.2, signed margin là -0.2 và loss 1.2. Ví dụ thứ hai bị phân loại sai và nhận penalty lớn.

## 4. Kernel: boundary phi tuyến mà không cần tính feature tường minh

Giả sử transformation $\phi(x)$ ánh xạ input vào feature space giàu hơn. Linear boundary trong không gian đó có thể phi tuyến theo input gốc. Kernel method dùng inner product:

$$
K(x,z)=\phi(x)^T\phi(z)
$$

mà không cần tính tường minh mọi feature đã biến đổi. Kernel phổ biến:

- Linear: $K(x,z)=x^Tz$.
- Polynomial: $K(x,z)=(\gamma x^Tz+r)^p$.
- Radial basis function: $K(x,z)=\exp(-\gamma\|x-z\|_2^2)$.

Tham số RBF $\gamma$ kiểm soát tính cục bộ. $\gamma$ quá lớn tạo boundary rất cục bộ, ngoằn ngoèo; $\gamma$ quá nhỏ làm biên trơn hơn và có thể underfit. Tune kernel, $C$ và $\gamma$ bằng validation, trong pipeline scale feature bên trong từng fold.

## 5. Multi-class prediction và xác suất

SVM cơ bản là binary. Package multi-class thường kết hợp classifier one-versus-rest hoặc one-versus-one. Raw decision score không phải xác suất calibrated. Nếu hệ thống phía sau cần probability, dùng calibration fit bằng validation data rồi đánh giá calibration riêng với chất lượng ranking.

## 6. Khi nào dùng SVM

SVM có thể hiệu quả cho dataset feature nhỏ đến trung bình, đặc biệt với representation và margin phù hợp. Kernel SVM có thể khó mở rộng theo số ví dụ và khó triển khai ở quy mô rất lớn. Với dataset lớn, linear model, tree ensemble hoặc neural network có thể thực tế hơn. So sánh với baseline đơn giản bằng split hợp lệ.

## 7. Bài luyện tập kèm đáp án

1. Với $w=(3,4)$, $b=0$ và $x=(0,2)$, distance tới boundary là bao nhiêu? **Đáp án:** $|8|/5=1.6$.
2. Điều gì xảy ra với soft-margin penalty khi $C$ rất lớn? **Đáp án:** margin violation trở nên đắt, nên model ưu tiên tránh chúng mạnh hơn.
3. Decision score của kernel SVM tự động là xác suất calibrated phải không? **Đáp án:** không.
4. Vì sao scale feature trước RBF-kernel SVM? **Đáp án:** Euclidean distance và kernel value phụ thuộc scale tọa độ.

## Lỗi thường gặp

- Quên mẫu số $\|w\|$ khi diễn giải geometric distance.
- Giả định hard-margin luôn khả thi trên dữ liệu thực nhiều noise.
- Tune kernel trên test set.
- So sánh giá trị $C$ giữa objective chuẩn hóa khác nhau như thể chúng giống hệt.
- Coi decision score là probability.
