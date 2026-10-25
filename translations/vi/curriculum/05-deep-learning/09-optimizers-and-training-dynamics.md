# Optimizer và động lực training

[Xem bài gốc bằng tiếng Anh](../../../../curriculum/05-deep-learning/09-optimizers-and-training-dynamics.md)

Optimizer là quy tắc dùng gradient để thay đổi parameter. Nó không quyết định label đúng không, split có hợp lệ không hay model có generalize không. Hãy xem optimization là một phần của thí nghiệm, không phải thứ thay thế thí nghiệm.

## Mục tiêu học tập

- Suy ra update của stochastic gradient descent và momentum.
- Giải thích ước lượng moment thứ nhất/thứ hai của Adam và bias correction.
- Phân biệt L2 regularization với weight decay tách rời trong adaptive optimizer.
- Chẩn đoán learning rate quá lớn hoặc quá nhỏ.
- Xây thí nghiệm training có kiểm soát và diễn giải curve.

## Điều kiện tiên quyết

Nắm chain rule và gradient descent. Hoàn thành [backpropagation](05-backpropagation-worked-example.md) và [thực hành training deep network](03-training-deep-networks-in-practice.md).

## 1. Full-batch và stochastic gradient

Với dataset loss $J(\theta)=\frac1N\sum_{i=1}^{N}\ell_i(\theta)$, full-batch gradient descent dùng mọi training example:

$$
\theta_{t+1}=\theta_t-\eta\nabla_\theta J(\theta_t).
$$

Stochastic gradient descent (SGD) ước lượng gradient bằng một example hoặc minibatch $B_t$:

$$
g_t=\frac1{|B_t|}\sum_{i\in B_t}\nabla_\theta\ell_i(\theta_t),
\qquad
\theta_{t+1}=\theta_t-\eta g_t.
$$

Minibatch giúp mỗi update rẻ hơn nhưng thêm sampling noise. Noise có thể hữu ích nhưng làm loss curve dao động. Batch size ảnh hưởng memory, throughput, gradient noise và đôi khi cả learning rate tốt nhất.

## 2. Momentum

Momentum giữ trung bình mũ có trọng số của gradient gần đây:

$$
v_t=\beta v_{t-1}+g_t,\qquad
\theta_t=\theta_{t-1}-\eta v_t.
$$

Một số implementation dùng $(1-\beta)g_t$ trong định nghĩa velocity; cách đó scale state khác và cần cách hiểu learning rate tương thích. Luôn xác định convention trước khi so công thức.

Khi gradient liên tục cùng hướng, velocity tích lũy khiến di chuyển nhanh hơn. Khi gradient đổi hướng liên tục, phép trung bình làm dịu dao động. Momentum coefficient lớn cũng khiến optimizer phản ứng chậm hơn khi hướng hữu ích đổi.

## 3. Adam

Adam theo dõi trung bình mũ của gradient và bình phương từng phần tử gradient:

$$
m_t=\beta_1m_{t-1}+(1-\beta_1)g_t,
\qquad
v_t=\beta_2v_{t-1}+(1-\beta_2)g_t^2.
$$

Vì hai state bắt đầu bằng 0, giá trị đầu bị lệch về 0. Giá trị hiệu chỉnh bias:

$$
\hat m_t=\frac{m_t}{1-\beta_1^t},
\qquad
\hat v_t=\frac{v_t}{1-\beta_2^t}.
$$

Update:

$$
\theta_t=\theta_{t-1}-\eta
\frac{\hat m_t}{\sqrt{\hat v_t}+\epsilon}.
$$

Căn bậc hai, chia và $\epsilon$ được áp dụng từng phần tử. Ước lượng moment hai điều chỉnh scale theo parameter. Adam không được bảo đảm tốt nhất cho mọi dataset/architecture; hãy so với baseline có ý nghĩa như SGD kèm momentum.

## 4. Tự tính một bước Adam

Với một scalar parameter, đặt $\theta_0=1$, gradient $g_1=2$, $\beta_1=0.9$, $\beta_2=0.999$, $\eta=0.1$, bỏ qua $\epsilon$ rất nhỏ để tính:

$$
m_1=0.1(2)=0.2,\quad
v_1=0.001(2^2)=0.004,
$$

$$
\hat m_1=\frac{0.2}{0.1}=2,\qquad
\hat v_1=\frac{0.004}{0.001}=4.
$$

Vậy bước update là $0.1(2/\sqrt4)=0.1$, nên $\theta_1=0.9$. Bias correction quan trọng: nếu bỏ qua, bước đầu nhỏ hơn nhiều.

## 5. Adam và AdamW

Thêm L2 penalty vào objective làm đổi gradient:

$$
J_{\mathrm{reg}}(\theta)=J(\theta)+\frac{\lambda}{2}\|\theta\|_2^2,
\qquad
\nabla J_{\mathrm{reg}}=\nabla J+\lambda\theta.
$$

Với SGD thuần, cách này gần giống nhân parameter với hệ số shrinkage mỗi bước. Với adaptive optimizer, đưa $\lambda\theta$ vào gradient cũng làm đổi moment estimate. Decoupled weight decay áp dụng shrinkage riêng:

$$
\theta_t=\theta_{t-1}
-\eta\frac{\hat m_t}{\sqrt{\hat v_t}+\epsilon}
-\eta\lambda\theta_{t-1}.
$$

Đây là khác biệt cốt lõi của AdamW. Quy ước parameter nào bị decay — thường loại bias hoặc normalization parameter — cần được nêu và validation.

## 6. Learning-rate schedule

Learning rate điều khiển scale update. Schedule phổ biến:

- warm-up ngắn nếu update ban đầu bất ổn ở rate mục tiêu;
- step decay ở milestone định trước;
- cosine decay theo training budget cố định;
- giảm rate khi validation performance ngừng cải thiện.

Schedule là một phần training procedure. Hãy ghi nó lại, vì “cùng model và optimizer” không phải so sánh tái lập nếu schedule khác nhau.

## 7. Gradient clipping

Nếu global gradient norm vượt ngưỡng c, norm clipping rescale gradient:

$$
g\leftarrow g\cdot\min\left(1,\frac{c}{\|g\|_2}\right).
$$

Clipping giới hạn update lớn bất thường; không sửa loss sai, data kém hay learning rate không phù hợp. Log tần suất clipping. Nếu gần bước nào cũng kích hoạt, hãy tìm nguyên nhân.

## 8. Đọc curve trước khi đổi model

| Quan sát | Kiểm tra trước |
| --- | --- |
| Train và validation loss đều còn cao | label, preprocessing, capacity, learning rate |
| Train loss giảm, validation loss tăng | overfitting, split không leakage, regularization, lệch dữ liệu |
| Loss thành NaN hoặc nhảy mạnh | learning rate, input không hữu hạn, độ ổn định loss/logit, gradient norm |
| Loss hầu như không đổi | parameter bị đóng băng, gradient 0, target sai, rate quá thấp |
| Train cải thiện nhưng task metric không đổi | loss/metric không khớp, threshold, class balance, calibration |

Mỗi lần chỉ đổi một yếu tố và giữ lại run đã biết là tốt. Vẽ cả loss lẫn task metric; objective training không phải lúc nào cũng là objective deployment.

## 9. Thí nghiệm optimizer có kiểm soát

1. Cố định data split, initialization, model, batch size và số update.
2. Thử dải learning rate logarithmic nhỏ.
3. Chỉ so SGD momentum với AdamW sau khi pipeline overfit được tiny batch.
4. Tách lựa chọn theo validation khỏi báo cáo test cuối.
5. Ghi optimizer setting, seed, hardware, precision và checkpoint validation tốt nhất.
6. Nếu compute cho phép, lặp setting triển vọng với nhiều seed; báo độ biến thiên.

Đừng chỉ so seed tốt nhất. Phương pháp thắng một lần nhưng thường xuyên thất bại có thể kém hữu ích hơn phương pháp hơi yếu nhưng ổn định.

## Bài luyện tập kèm đáp án

1. Vì sao Adam dùng bias correction?
2. Trong ví dụ một bước, vì sao độ lớn update đã hiệu chỉnh không phụ thuộc độ lớn gradient với scalar dương này?
3. Vì sao thêm L2 vào gradient không giống hệt decoupled weight decay trong Adam?
4. Cần kiểm tra gì nếu training loss thành NaN?
5. Vì sao learning rate thường là optimizer parameter đầu tiên cần tune?

### Đáp án

1. Moving average khởi đầu từ 0 nên giá trị đầu đánh giá thấp moment; correction bù lệch do khởi tạo.
2. Với một scalar gradient dương, hai moment đầu sau correction là g và $g^2$, tỷ lệ thành $g/|g|=1$. Update bằng learning rate; điều này không có nghĩa training nhiều chiều/nhiễu về sau bất biến scale.
3. Thêm L2 đổi gradient đi vào moment estimate; decoupled decay shrink parameter tách khỏi estimate.
4. Kiểm tra input/target có hữu hạn không, logit, loss computation, learning rate, gradient norm và mixed-precision scaling trước khi tăng complexity.
5. Nó nhân trực tiếp mọi update, có thể chuyển training từ đứng yên sang bất ổn. Setting khác khó diễn giải trước khi step scale hợp lý.

## Lỗi thường gặp

- Nghĩ Adam không cần tune learning rate.
- Nhầm train-loss minimum với checkpoint validation tốt nhất.
- Gradient clip mà không theo dõi tần suất kích hoạt.
- So optimizer với update budget hoặc batch size khác nhau.
- Gọi AdamW là “Adam cộng L2” mà không giải thích decoupling.
