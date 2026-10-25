# Phân phối xác suất và Bayes

> Bản tiếng Anh: [Probability Distributions and Bayes](../../../../curriculum/01-math/07-probability-distributions-and-bayes.md)

Bài học này xây dựng các công cụ xác suất được dùng trong classifier, ước lượng độ bất định, generative model và statistical learning. Mỗi công thức cần nêu rõ đại lượng nào ngẫu nhiên, quan sát nào đã biết và điều kiện nào đang được xét.

## Mục tiêu học tập

Sau bài này, bạn có thể:

- phân biệt random variable với một giá trị đã quan sát;
- tính joint, marginal và conditional probability;
- dùng Bayes' rule mà không đảo nhầm điều kiện;
- nhận biết Bernoulli, categorical, binomial và Gaussian distribution;
- liên hệ expected value và variance với thống kê dữ liệu và output mô hình.

## 1. Biến cố, random variable và distribution

**Event (biến cố)** là tập hợp kết quả có thể xảy ra, ví dụ “đồng xu ra mặt ngửa”. **Random variable (biến ngẫu nhiên)** ánh xạ mỗi kết quả thành một con số. Chữ hoa $X$ thường đặt tên cho random variable; chữ thường $x$ là một giá trị mà biến có thể nhận. **Distribution (phân phối)** gán xác suất hoặc mật độ xác suất cho các giá trị có thể có của X.

Với random variable rời rạc, tổng xác suất bằng một:

$$
\sum_x P(X=x)=1.
$$

Với biến liên tục, xác suất được biểu diễn bằng density $p(x)$ và tích phân của density bằng một. Giá trị density tự nó không phải xác suất; xác suất của một khoảng là diện tích dưới đường density trên khoảng đó.

## 2. Joint, marginal, conditional probability và độc lập

Joint probability $P(X=x,Y=y)$ mô tả hai sự kiện cùng xảy ra. Marginal probability của X được tính bằng cách cộng qua mọi giá trị có thể của Y:

$$
P(X=x)=\sum_y P(X=x,Y=y).
$$

Với hai event A và B, conditional probability là:

$$
P(A\mid B)=\frac{P(A\cap B)}{P(B)}, \qquad P(B)>0.
$$

Hai event độc lập nếu biết event này không làm thay đổi xác suất event kia:

$$
P(A\cap B)=P(A)P(B), \qquad P(A\mid B)=P(A).
$$

Độc lập là một giả định, không đồng nghĩa với “hai cột khác nhau”. Hai phép đo từ cùng một người, các frame liền kề trong video và nhiều sự kiện từ cùng khách hàng thường phụ thuộc nhau.

### Ví dụ có lời giải: xét nghiệm và bệnh hiếm

Giả sử 1% dân số mắc bệnh. Xét nghiệm có sensitivity 90% và false-positive rate 5%. Xét 10.000 người:

- 100 người mắc bệnh; khoảng 90 người có kết quả dương tính.
- 9.900 người không mắc; khoảng 495 người dương tính nhầm.
- Trong 585 ca dương tính, có 90 true positive.

Vì vậy:

$$
P(\text{mắc bệnh}\mid +)=\frac{90}{90+495}\approx 0.154.
$$

Sensitivity 90% là $P(+\mid\text{mắc bệnh})$, không phải $P(\text{mắc bệnh}\mid +)$. Base rate làm thay đổi kết quả.

## 3. Bayes' rule

Bayes' rule cập nhật prior belief sau khi quan sát bằng chứng:

$$
P(A\mid B)=\frac{P(B\mid A)P(A)}{P(B)}.
$$

Trong đó $P(A)$ là prior, $P(B\mid A)$ là likelihood của bằng chứng khi A đúng và $P(A\mid B)$ là posterior. Với các khả năng loại trừ lẫn nhau $A_1,\ldots,A_K$:

$$
P(B)=\sum_{k=1}^{K}P(B\mid A_k)P(A_k).
$$

Naive Bayes dùng Bayes' rule và xấp xỉ likelihood của vector feature theo lớp bằng tích các likelihood riêng của từng feature. Giả định độc lập có điều kiện này thường không đúng hoàn toàn, nhưng classifier vẫn có thể là baseline nhanh và mạnh.

## 4. Kỳ vọng và phương sai

Kỳ vọng là trung bình có trọng số xác suất:

$$
\mathbb{E}[X]=\sum_x xP(X=x)
$$

với biến rời rạc, hoặc $\mathbb{E}[X]=\int xp(x)\,dx$ với biến liên tục. Variance là bình phương khoảng cách trung bình đến kỳ vọng:

$$
\operatorname{Var}(X)=\mathbb{E}\left[(X-\mathbb{E}[X])^2\right]
=\mathbb{E}[X^2]-\mathbb{E}[X]^2.
$$

Standard deviation $\sigma=\sqrt{\operatorname{Var}(X)}$ có cùng đơn vị với X. Với hằng số a và random variable X:

$$
\mathbb{E}[aX+b]=a\mathbb{E}[X]+b,
\qquad
\operatorname{Var}(aX+b)=a^2\operatorname{Var}(X).
$$

Expected value là trung bình lý thuyết theo distribution; sample mean $\bar{x}$ là ước lượng từ dữ liệu quan sát.

## 5. Các distribution thường gặp

### Bernoulli: một kết quả có/không

Nếu $X\in\{0,1\}$ và $P(X=1)=p$ thì $X\sim\operatorname{Bernoulli}(p)$:

$$
P(X=x)=p^x(1-p)^{1-x}, \qquad
\mathbb{E}[X]=p, \qquad
\operatorname{Var}(X)=p(1-p).
$$

Đây là distribution cơ bản cho một label nhị phân hoặc một lần tung đồng xu.

### Categorical: một trong nhiều lớp

Với K lớp có xác suất $p_1,\ldots,p_K$, mỗi $p_k\geq0$ và $\sum_kp_k=1$. Target one-hot có đúng một phần tử bằng 1; categorical probability là xác suất của lớp thật. Softmax model chuyển K logits thành vector xác suất categorical.

### Binomial: số lần thành công qua các lượt thử

Với n phép thử Bernoulli độc lập và cùng xác suất thành công p, tổng S tuân theo $\operatorname{Binomial}(n,p)$:

$$
P(S=k)=\binom{n}{k}p^k(1-p)^{n-k},
\qquad
\mathbb{E}[S]=np,
\qquad
\operatorname{Var}(S)=np(1-p).
$$

Các giả định quan trọng: mỗi lượt độc lập và có cùng p.

### Gaussian (normal): distribution liên tục dạng chuông

Gaussian một chiều có mean $\mu$ và variance $\sigma^2$:

$$
p(x)=\frac{1}{\sqrt{2\pi\sigma^2}}
\exp\left(-\frac{(x-\mu)^2}{2\sigma^2}\right).
$$

Mean xác định tâm; variance quyết định độ phân tán. Gaussian noise dẫn đến squared-error loss khi fit mô hình bằng maximum likelihood. Multivariate Gaussian dùng vector mean và covariance matrix $\Sigma$; phần tử ngoài đường chéo mô tả quan hệ tuyến tính giữa các chiều.

## 6. Law of large numbers và lấy mẫu

Với mẫu độc lập, cùng distribution và có mean hữu hạn, sample average tiến đến kỳ vọng thật khi số mẫu tăng:

$$
\bar{X}_n=\frac{1}{n}\sum_{i=1}^n X_i \longrightarrow \mathbb{E}[X].
$$

Điều này không sửa được lấy mẫu thiên lệch, distribution shift hoặc sự phụ thuộc giữa ví dụ. Một triệu ví dụ lấy từ sai population vẫn có thể ước lượng sai đại lượng, chỉ là rất chính xác.

## 7. Luyện tập

1. Tung xúc xắc cân bằng. Viết $P(X=3)$ và $\mathbb{E}[X]$.
2. Nếu $P(A)=0.2$, $P(B\mid A)=0.5$, $P(B\mid\neg A)=0.1$, tính $P(A\mid B)$.
3. Binary model trả xác suất $p=0.7$. Nó gán xác suất nào cho lớp 1 và lớp 0? Distribution nào mô tả một label này?
4. Giải thích vì sao $\operatorname{Var}(2X+3)=4\operatorname{Var}(X)$.

**Đáp án kiểm tra:** $1/6$ và $3.5$; $0.2(0.5)/(0.2(0.5)+0.8(0.1))=0.556$; Bernoulli với xác suất 0.7 và 0.3; cộng hằng số không đổi độ phân tán, nhân giá trị với 2 làm độ lệch tăng gấp đôi nên bình phương độ lệch tăng gấp bốn.

## Lỗi thường gặp

- Đảo $P(A\mid B)$ và $P(B\mid A)$.
- Gọi density là xác suất.
- Giả định các cột feature độc lập chỉ vì tên khác nhau.
- Báo cáo mean mà không có độ bất định, độ phân tán hoặc mô tả mẫu.
