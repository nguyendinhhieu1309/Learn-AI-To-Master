# Vector và ma trận

> Bản tiếng Anh: [Vectors and Matrices](../../../../curriculum/01-math/01-vectors-and-matrices.md)

## Mục tiêu học tập

Bạn sẽ học cách biểu diễn dữ liệu bằng vector và ma trận, cách tích vô hướng tạo ra điểm số hữu ích và vì sao kích thước tensor cần được theo dõi trong code mô hình.

## 1. Vector biểu diễn một danh sách có ý nghĩa

Vector là một danh sách có thứ tự gồm các số. Feature của một người có thể là x = [tuổi, chiều cao, cân nặng]. Thứ tự rất quan trọng: [tuổi, chiều cao] không giống [chiều cao, tuổi].

**Shape** cho biết số phần tử. Trong bài học này, vector cột được viết:

$$
x = \begin{bmatrix}2\\3\\4\end{bmatrix}, \qquad x \in \mathbb{R}^{3}.
$$

Tập số thực $\mathbb{R}$ cho biết giá trị là số thực; số mũ 3 cho biết có ba tọa độ.

## 2. Tích vô hướng: điểm có trọng số và độ tương đồng

Với hai vector cùng chiều dài, tích vô hướng nhân các phần tử tương ứng rồi cộng lại:

$$
w^T x = \sum_{j=1}^{d} w_j x_j.
$$

Nếu $w=[2,-1]$ và $x=[3,4]$ thì $w^Tx=2(3)-1(4)=2$. Linear model cộng thêm bias: $\hat y=w^Tx+b$.

Tích vô hướng cũng liên hệ với góc giữa hai vector:

$$
u^Tv=\|u\|_2\|v\|_2\cos\theta.
$$

Sau khi chuẩn hóa các vector về độ dài 1, tích vô hướng (cosine similarity) bằng 1 nếu cùng hướng, 0 nếu vuông góc và $-1$ nếu ngược hướng.

## 3. Norm đo độ lớn

Norm Euclid, hay $L_2$, được tính:

$$
\|x\|_2=\sqrt{\sum_{j=1}^{d}x_j^2}.
$$

Với $x=[3,4]$, norm bằng 5. Norm $L_1$ là $\|x\|_1=\sum_j|x_j|$, bằng 7 với [3,4]. Norm có thể biểu diễn khoảng cách hoặc ràng buộc trọng số mô hình. Regularization $L_1$ thường khiến một số trọng số bằng 0 chính xác; $L_2$ thường làm các trọng số co nhỏ dần.

## 4. Ma trận biến đổi và gom dữ liệu

Ma trận là một lưới hình chữ nhật. Nếu X có n hàng (ví dụ) và d cột (feature), shape của X là (n,d). Nhân Xw với w có shape (d,) tạo ra n điểm số, mỗi điểm ứng với một ví dụ.

Phép nhân ma trận chỉ hợp lệ khi hai kích thước trong khớp nhau: (n,d) @ (d,k) -> (n,k). Lỗi phổ biến là nhân shape không tương thích hoặc chuyển vị mảng không cần thiết. Hãy kiểm tra shape thay vì đoán.

Ma trận có thể biểu diễn phép biến đổi tuyến tính: xoay, kéo giãn, phản chiếu hoặc chiếu vector. Phép xoay đổi hướng nhưng giữ nguyên độ dài; phép chiếu chỉ giữ lại thành phần theo một số hướng.

## 5. Eigenvector và trực giác về PCA

Eigenvector của ma trận vuông là hướng mà ma trận chỉ co giãn, không đổi hướng:

$$
Av=\lambda v.
$$

Principal Component Analysis (PCA) tìm các hướng nắm bắt nhiều phương sai trong dữ liệu. PCA có thể nén dữ liệu, nhưng các principal component là tổ hợp feature mới và có thể khó diễn giải hơn. Khi dùng trước bước đánh giá mô hình, PCA phải được fit trên tập train.

## Ví dụ có lời giải: dự đoán theo một batch

Giả sử có 3 ví dụ, mỗi ví dụ gồm 2 feature:

$$
X = \begin{bmatrix}1&2\\0&1\\3&1\end{bmatrix}, \quad w=\begin{bmatrix}2\\-1\end{bmatrix}, \quad b=0.5.
$$

Điểm số là [1(2)+2(-1), 0(2)+1(-1), 3(2)+1(-1)] + 0.5 = [0.5,-0.5,5.5]. Shape là (3,2) × (2,) -> (3,).

## Tự kiểm tra

1. Tích vô hướng của [1,2] và [3,4] bằng bao nhiêu? **Đáp án:** 11.
2. Shape của 12 ví dụ với 5 feature là gì? **Đáp án:** (12,5).
3. Nếu A có shape (8,4), B có shape (4,2), shape của AB là gì? **Đáp án:** (8,2).

## Lỗi thường gặp

- Quên thứ tự feature phải giống nhau giữa huấn luyện và dự đoán.
- Nhầm phép nhân từng phần tử với phép nhân ma trận.
- So sánh độ lớn vector khi mục tiêu là so hướng bằng cosine, hoặc ngược lại.
- Fit phép giảm chiều trên toàn bộ dữ liệu trước khi chia train/test.
