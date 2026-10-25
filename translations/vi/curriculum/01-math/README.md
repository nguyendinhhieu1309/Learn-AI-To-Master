# Toán cho AI

> Bản tiếng Anh: [Mathematics for AI](../../../../curriculum/01-math/README.md)

## Kết quả cần đạt

- Đọc được biểu thức vector và ma trận dùng trong Machine Learning.
- Liên hệ đạo hàm và gradient với quá trình tối ưu mô hình.
- Dùng xác suất và thống kê để suy luận về độ bất định và bằng chứng.

## Kiến thức tiên quyết

Chỉ cần số học và đại số cơ bản để bắt đầu. Ôn lại phần còn thiếu khi gặp trong bài; có thể học module này song song với Python.

## Thứ tự gợi ý

1. [Đại số, hàm số và ký hiệu](00-algebra-functions-notation.md)
2. [Vector và ma trận](01-vectors-and-matrices.md)
3. [Giải tích và tối ưu hóa](02-calculus-and-optimization.md)
4. [Xác suất và thống kê](03-probability-and-statistics.md)
5. [Suy luận thống kê và thiết kế thí nghiệm](04-statistical-inference-and-experiment-design.md)
6. [Đại số tuyến tính cho Machine Learning](05-linear-algebra-for-machine-learning.md)
7. [Lý thuyết thông tin và tính toán số](06-information-theory-and-numerical-computation.md)
8. [Các phân phối xác suất và Bayes](07-probability-distributions-and-bayes.md)
9. [Likelihood, MLE và MAP](08-likelihood-mle-and-map.md)
10. [Tối ưu lồi và ràng buộc](09-convex-optimization-and-constraints.md)

Sau các bài học, làm [bộ bài tập có lời giải](PROBLEM_SET_WITH_SOLUTIONS.md) để tự kiểm tra khả năng suy ra và tính kết quả cốt lõi mà không sao chép ví dụ.

### Cách học từng bài

Với mỗi công thức hiển thị, trước tiên hãy xác định đầu vào, đầu ra, tham số và shape của từng ký hiệu. Tự tính ví dụ số bằng tay, diễn đạt kết quả bằng lời, rồi mới chuyển sang NumPy hoặc PyTorch. Ghi lại công thức bạn có thể tự suy ra và các giả định cần kiểm tra; chỉ nhớ công thức mà không hiểu kích thước và giả định là chưa đủ.

Không cần học hết mọi chủ đề nâng cao trước khi bắt đầu ML. Học xong bài 00–03 trước; sau đó học bài 04–09 song song với đánh giá mô hình và các thuật toán sử dụng chúng.

## Luyện tập

- Tính tích vô hướng và mô tả điều gì xảy ra khi nhân vector với một hệ số.
- Lấy đạo hàm hàm bậc hai đơn giản và dùng gradient descent để tiến đến cực tiểu.
- Mô phỏng tung đồng xu, ước lượng xác suất và quan sát ước lượng thay đổi theo kích thước mẫu.
- So sánh mean và median trên dữ liệu có outlier.
- Suy ra gradient của squared error cho linear model một feature rồi kiểm tra một thành phần bằng finite difference.
- Tính maximum likelihood estimate cho Bernoulli từ số lần tung đồng xu, sau đó giải thích prior Beta làm thay đổi ước lượng ra sao.

## Dự án kiểm tra

Tự cài đặt linear regression cho bộ dữ liệu rất nhỏ bằng nghiệm dạng đóng và gradient descent. Vẽ loss theo quá trình huấn luyện, so sánh hai kết quả và giải thích scale feature ảnh hưởng đến tối ưu hóa thế nào.

## Lỗi thường gặp

- Coi ma trận như danh sách số không kiểu: cần theo dõi kích thước và ý nghĩa mỗi trục.
- Nhầm tương quan với quan hệ nhân quả.
- Hiểu đạo hàm là điểm số mô hình thay vì tốc độ thay đổi cục bộ.
- Dùng công thức mà không kiểm tra giả định.
- Nhầm xác suất có điều kiện $P(A\mid B)$ với chiều ngược lại $P(B\mid A)$.
- Làm mất phép chuyển vị hoặc batch axis khi chuyển phương trình vô hướng thành phương trình ma trận.
- Coi đạo hàm là thay đổi chính xác cho bước lớn thay vì xấp xỉ cục bộ.
