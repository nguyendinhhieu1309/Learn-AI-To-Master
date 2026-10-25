# Đại số, hàm số và ký hiệu

> Bản tiếng Anh: [Algebra, Functions, and Notation](../../../../curriculum/01-math/00-algebra-functions-notation.md)

## Mục tiêu học tập

Sau bài này, bạn có thể đọc ký hiệu Toán thường gặp trong bài báo ML, thay số vào hàm và theo dõi ý nghĩa của từng ký hiệu.

## 1. Biến và hàm số

**Biến** là tên đại diện cho một giá trị. **Hàm số** nhận đầu vào và trả ra đầu ra. Ví dụ, nếu giá cước taxi gồm phí cố định và phí theo kilomet, ta viết:

$$
f(x) = 3 + 2x,
$$

trong đó $x$ là quãng đường tính bằng kilomet và $f(x)$ là tiền cước. Với 4 km, $f(4)=3+2(4)=11$.

Trong ML, mô hình cũng là một hàm. Mô hình nhận các feature $x$ và tạo dự đoán $f(x)$. Quá trình huấn luyện thay đổi các tham số, thường ký hiệu $w$ hoặc $\theta$, để dự đoán khớp với các ví dụ hơn.

## 2. Phương trình và biến đổi

Phương trình khẳng định hai biểu thức có cùng giá trị. Để giải $2x+3=11$, đảo ngược thứ tự phép toán: trừ 3 rồi chia 2, được $x=4$. Cùng một phép toán phải áp dụng cho cả hai vế.

Điều này quan trọng khi đọc công thức. Nếu $\hat y=wx+b$, dự đoán $\hat y$ thay đổi theo đầu vào $x$; $w$ quyết định độ dốc, còn $b$ là giá trị tại $x=0$.

## 3. Lũy thừa, căn và logarithm

- $x^2$ nghĩa là $x\cdot x$; $\sqrt{x}$ là số không âm mà bình phương bằng $x$.
- $a^b$ là nhân $a$ với chính nó $b$ lần khi $b$ là số nguyên dương; các quy tắc lũy thừa mở rộng ý tưởng này.
- $\log(x)$ đảo ngược phép lũy thừa. Với logarithm tự nhiên, $\ln(e^x)=x$.

Logarithm biến phép nhân thành phép cộng: $\ln(ab)=\ln(a)+\ln(b)$. Trong Machine Learning, điều này hữu ích vì tích các xác suất có thể rất nhỏ. Ta thường cực đại hóa log-probability thay vì probability vì tính toán ổn định hơn và đạt cùng điểm cực đại.

## 4. Tổng và trung bình

Ký hiệu $\sum_{i=1}^{n}x_i$ nghĩa là cộng $x_1+x_2+\cdots+x_n$. Trung bình là:

$$
\bar{x} = \frac{1}{n}\sum_{i=1}^{n} x_i.
$$

Với các giá trị 2, 4, 9, tổng là 15 và mean là 5. Nhiều objective huấn luyện là trung bình loss trên từng ví dụ:

$$
J(\theta)=\frac{1}{n}\sum_{i=1}^{n}\ell_i(\theta).
$$

## 5. Shape, đơn vị và chỉ số dưới

Ký hiệu là ngôn ngữ viết gọn. Chỉ số dưới thường xác định ví dụ: $x_i$ là ví dụ thứ $i$. Chỉ số trên có thể là lũy thừa ($x^2$) hoặc layer ($h^{(2)}$); hãy kiểm tra định nghĩa của tác giả. Vector có 3 phần tử có shape $(3,)$; ma trận có 5 hàng, 3 cột có shape $(5,3)$.

Gắn ý nghĩa và đơn vị cho giá trị. Nếu feature tính bằng centimet nhưng công thức cần mét, kết quả có thể sai 100 lần. Trong code, tên như distance_m giúp tránh lỗi đơn vị.

## Ví dụ có lời giải

Một mô hình dự đoán giá nhà theo $p=50+4a$, trong đó $a$ là diện tích tính theo đơn vị 10 mét vuông và $p$ là nghìn đô la. Với nhà 70 mét vuông, $a=7$, nên $p=50+4(7)=78$ nghìn đô la. Hệ số góc cho biết dự đoán tăng 4 nghìn đô với mỗi 10 mét vuông tăng thêm. Điều này **không** chứng minh diện tích tăng gây ra mức tăng giá đó; mô hình chỉ mô tả mối quan hệ đã khớp.

## Tự kiểm tra

1. Nếu $g(x)=5x-2$, $g(3)$ bằng bao nhiêu?
2. Viết trung bình của năm giá trị bằng ký hiệu tổng.
3. Dataset có 100 ví dụ và 8 feature. Feature matrix có thể có shape nào?

**Đáp án:** 13; $\frac15\sum_{i=1}^{5}x_i$; $(100,8)$ nếu hàng là ví dụ và cột là feature.

## Lỗi thường gặp

- Nghĩ rằng một ký hiệu luôn có cùng ý nghĩa trong mọi sách hoặc bài báo.
- Quên vector đang được coi là hàng hay cột.
- Nhầm $\log(x^2)$ với $(\log x)^2$.
- Bỏ qua đơn vị hoặc số ví dụ mà phép trung bình đại diện.
