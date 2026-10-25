# Likelihood, Maximum Likelihood và MAP

> Bản tiếng Anh: [Likelihood, Maximum Likelihood, and MAP](../../../../curriculum/01-math/08-likelihood-mle-and-map.md)

Likelihood chuyển các giả định xác suất của mô hình thành đại lượng có thể fit từ dữ liệu. Đây là cầu nối từ probability đến các loss như squared error và cross-entropy.

## Mục tiêu học tập

- Phân biệt probability distribution với likelihood function.
- Suy ra maximum-likelihood estimate cho xác suất Bernoulli.
- Giải thích vì sao quan sát độc lập tạo tổng log-likelihood.
- Liên hệ training loss thường gặp với giả định likelihood.
- Giải thích prior dẫn từ maximum likelihood đến MAP và regularization thế nào.

## 1. Probability và likelihood dùng cùng biểu thức theo hai cách

Giả sử quan sát $D=(x_1,\ldots,x_n)$ được mô hình hóa bằng tham số $\theta$. Xác suất dữ liệu khi tham số cố định là:

$$
p(D\mid\theta).
$$

Sau khi dữ liệu đã quan sát và cố định, xem cùng biểu thức như hàm theo $\theta$ thì đó là **likelihood**:

$$
L(\theta;D)=p(D\mid\theta).
$$

Likelihood nói chung không phải probability distribution trên các giá trị tham số $\theta$; các giá trị không cần cộng hay tích phân thành 1. Nó cho biết tham số nào làm dữ liệu quan sát tương đối hợp lý hơn.

## 2. Maximum likelihood estimation (MLE)

MLE chọn tham số làm likelihood lớn nhất:

$$
\hat{\theta}_{\mathrm{MLE}}=\arg\max_\theta p(D\mid\theta).
$$

Với các quan sát độc lập có điều kiện:

$$
p(D\mid\theta)=\prod_{i=1}^{n}p(x_i\mid\theta).
$$

Tích của nhiều xác suất nhỏ có thể underflow. Logarithm là hàm tăng nên giữ nguyên điểm cực đại, đồng thời đổi tích thành tổng:

$$
\hat{\theta}_{\mathrm{MLE}}
=\arg\max_\theta \sum_{i=1}^{n}\log p(x_i\mid\theta)
=\arg\min_\theta -\sum_{i=1}^{n}\log p(x_i\mid\theta).
$$

Negative log-likelihood (NLL) vì vậy là loss tự nhiên. Chia cho n thay đổi scale nhưng không đổi nghiệm tối thiểu.

## 3. Ví dụ: ước lượng xác suất mặt ngửa

Đặt $x_i=1$ nếu ngửa, 0 nếu sấp. Giả sử các lần tung độc lập, xác suất ngửa chưa biết là p. Likelihood:

$$
L(p;D)=\prod_{i=1}^{n}p^{x_i}(1-p)^{1-x_i}
=p^h(1-p)^{n-h},
$$

trong đó $h=\sum_i x_i$ là số lần ngửa. Log-likelihood:

$$
\ell(p)=h\log p+(n-h)\log(1-p).
$$

Lấy đạo hàm và cho bằng 0:

$$
\frac{d\ell}{dp}=\frac{h}{p}-\frac{n-h}{1-p}=0
\quad\Longrightarrow\quad
\hat p_{\mathrm{MLE}}=\frac{h}{n}.
$$

Nếu 7 trong 10 lần tung ra ngửa thì $\hat p=0.7$. Chỉ 10 quan sát khiến ước lượng còn bất định; MLE tự nó không nói rõ lượng bằng chứng hỗ trợ ước lượng.

## 4. Vì sao xuất hiện các loss quen thuộc

### Squared error là Gaussian NLL

Giả sử target hồi quy bằng dự đoán mô hình cộng independent Gaussian noise có variance cố định $\sigma^2$:

$$
y_i\mid x_i,\theta\sim\mathcal{N}(f_\theta(x_i),\sigma^2).
$$

Bỏ qua các thành phần không phụ thuộc vào $\theta$, negative log-likelihood tỷ lệ với:

$$
\sum_i\left(y_i-f_\theta(x_i)\right)^2.
$$

Vì vậy tối thiểu hóa squared error tương ứng fit mô hình conditional-mean dưới giả định Gaussian noise. Nếu lỗi có heavy tail hoặc outlier thì giả định này có thể không phù hợp; MAE hoặc robust likelihood có thể tốt hơn.

### Binary cross-entropy là Bernoulli NLL

Với binary target $y_i\in\{0,1\}$ và xác suất dự đoán $p_i=P(y_i=1\mid x_i)$:

$$
P(y_i\mid x_i)=p_i^{y_i}(1-p_i)^{1-y_i}.
$$

Lấy negative logarithm được binary cross-entropy:

$$
\ell_i=-\left[y_i\log p_i+(1-y_i)\log(1-p_i)\right].
$$

Nếu $y_i=1$ thì loss là $-\log p_i$; nếu $y_i=0$ thì là $-\log(1-p_i)$. Dự đoán tự tin nhưng trái với label bị phạt lớn.

### Multiclass cross-entropy là categorical NLL

Với xác suất lớp $p_{i1},\ldots,p_{iK}$ và one-hot label $y_{ik}$:

$$
\ell_i=-\sum_{k=1}^{K}y_{ik}\log p_{ik}.
$$

Khi label one-hot, chỉ xác suất của lớp thật đóng góp vào loss. Vì vậy hàm loss ổn định trong phần mềm thường nhận trực tiếp logits và integer label.

## 5. MAP: likelihood cộng thông tin prior

Maximum a posteriori (MAP) chọn tham số có posterior probability cao nhất:

$$
\hat{\theta}_{\mathrm{MAP}}
=\arg\max_\theta p(\theta\mid D)
=\arg\max_\theta p(D\mid\theta)p(\theta),
$$

đẳng thức cuối bỏ mẫu số chuẩn hóa $p(D)$ vì nó không phụ thuộc $\theta$. Lấy negative logarithm:

$$
\hat{\theta}_{\mathrm{MAP}}
=\arg\min_\theta\left[-\log p(D\mid\theta)-\log p(\theta)\right].
$$

Prior thêm một khoản penalty:

- Gaussian mean 0 trên weight tạo penalty $L_2$, thường gọi là weight decay hoặc ridge regularization.
- Laplace centered tại 0 tạo penalty $L_1$, có thể làm weight thưa.

Độ mạnh của prior quyết định scale penalty. Chọn tham số này mà không dùng final test set.

## 6. Ví dụ prior Beta

Với coin flip, chọn prior $p\sim\operatorname{Beta}(\alpha,\beta)$. Sau h lần ngửa và t lần sấp, posterior là:

$$
p\mid D\sim\operatorname{Beta}(\alpha+h,\beta+t).
$$

Với uniform prior $\operatorname{Beta}(1,1)$ và 7 lần ngửa trong 10 lần tung, posterior mean:

$$
\mathbb{E}[p\mid D]=\frac{1+7}{1+1+10}=\frac{8}{12}\approx0.667.
$$

Ước lượng này được kéo nhẹ về tâm prior 0.5. Prior giúp ổn định ước lượng khi ít dữ liệu; prior chọn kém cũng có thể làm lệch kết quả, vì vậy cần nêu và giải thích nó.

## 7. Luyện tập

1. Đồng xu thiên lệch ra ngửa 12 lần trong 20 lần tung. Tìm MLE.
2. Nếu một quan sát rất khó xảy ra dưới mọi mô hình ứng viên, log-likelihood ảnh hưởng huấn luyện ra sao?
3. Giả định noise nào nối squared error với MLE?
4. Mô hình dự đoán 0.9 cho label positive đúng. Tính binary cross-entropy của ví dụ đó.
5. Giải thích bằng lời vì sao penalty $L_2$ tương ứng Gaussian prior.

**Đáp án kiểm tra:** 0.6; nó tạo log-likelihood âm lớn và loss cao; Gaussian conditional noise có variance cố định; $-\log(0.9)\approx0.105$; weight lớn ít có khả năng dưới Gaussian prior centered tại 0 nên optimizer bị hạn chế chọn chúng.

## Lỗi thường gặp

- Gọi likelihood là probability distribution trên tham số mô hình.
- Tối đa hóa trực tiếp tích xác suất thay vì dùng log-probability.
- Quên mỗi loss mã hóa giả định về target hoặc noise.
- Dùng prior hay regularizer mà không giải thích scale hoặc kiểm định trên dữ liệu.
