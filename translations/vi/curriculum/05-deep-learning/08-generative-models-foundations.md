# Nền tảng generative model

[Xem bài gốc bằng tiếng Anh](../../../../curriculum/05-deep-learning/08-generative-models-foundations.md)

Generative model học một phân phối hoặc quy trình tạo sample. Từ “generative” mô tả mục tiêu modeling; nó không có nghĩa kết quả đúng, hữu ích, nguyên bản hay an toàn.

## Mục tiêu học tập

- Phân biệt discriminative prediction với generative modeling.
- So sánh objective autoregressive, variational autoencoder, GAN và diffusion.
- Giải thích training và sampling của từng họ model.
- Nhận biết khoảng trống evaluation và rủi ro deployment thường gặp.

## 1. Câu hỏi discriminative và generative

Discriminative model ước lượng $P(y\mid x)$ hoặc decision boundary: đưa image vào thì có class nào? Generative model hướng tới biểu diễn $P(x)$ hoặc $P(x\mid c)$ rồi sample dữ liệu có vẻ hợp lý. Một số model kết hợp cả hai mục tiêu. Likelihood hay training score không đồng nghĩa chất lượng cảm nhận của người hay ích lợi downstream.

## 2. Autoregressive model

Chain rule phân tích phân phối sequence thành xác suất điều kiện từng bước:

$$
P(x_1,\ldots,x_T)=\prod_{t=1}^{T}P(x_t\mid x_{<t}).
$$

Training thường tối thiểu hóa next-token negative log-likelihood. Sampling diễn ra tuần tự: dự đoán phân phối token kế, chọn/sample token, thêm nó vào chuỗi rồi lặp. Cách này linh hoạt nhưng inference tuần tự; output có thể trôi lệch khi model dựa vào lỗi trước do chính nó tạo.

## 3. Variational autoencoder (VAE)

Encoder xấp xỉ latent posterior $q_\phi(z\mid x)$; decoder mô hình hóa $p_\theta(x\mid z)$. Training tối đa hóa evidence lower bound (ELBO):

$$
\log p_\theta(x)\geq
\mathbb{E}_{q_\phi(z\mid x)}[\log p_\theta(x\mid z)]
-D_{KL}(q_\phi(z\mid x)\|p(z)).
$$

Số hạng đầu khuyến khích tái dựng; KL term khuyến khích encoded distribution gần prior để có thể sinh sample mới từ prior. Prior normal chuẩn phổ biến nhưng không bắt buộc. VAE cung cấp objective latent-space có cấu trúc, dù pixel-level likelihood có thể không khớp độ nét cảm nhận hay chất lượng ngữ nghĩa.

## 4. Generative adversarial network (GAN)

Generator ánh xạ noise z thành sample G(z). Discriminator cố phân biệt sample thật và sample sinh ra. Objective đối kháng có thể tạo sample sắc nét nhưng training bất ổn; mode collapse làm giảm diversity; discriminator có thể quá mạnh hoặc quá yếu. Theo dõi cả sample quality lẫn coverage/diversity, không chỉ một ảnh bắt mắt.

## 5. Diffusion model

Forward diffusion thêm noise vào dữ liệu dần dần. Neural network học dự đoán hoặc loại noise tại nhiều thời điểm. Sampling bắt đầu từ noise rồi lặp quá trình đảo ngược đã học:

$$
x_T\sim\mathcal{N}(0,I),
\qquad x_{t-1}=\text{reverse-step}_\theta(x_t,t).
$$

Nhiều hệ thống training bằng noise-prediction objective như $\mathbb{E}\|\epsilon-\epsilon_\theta(x_t,t)\|_2^2$. Conditioning có thể hướng dẫn generation bằng text, class hay input khác. Sampling thường cần nhiều bước denoise; quality, tốc độ và diversity tùy model, sampler và guidance setting.

## 6. So sánh họ model

| Họ model | Training target thường gặp | Cách sampling | Thách thức phổ biến |
| --- | --- | --- | --- |
| Autoregressive | Next-token likelihood | Token lần lượt | Latency tuần tự, lỗi tích lũy |
| VAE | ELBO / reconstruction cộng KL | Sample latent rồi decode | Output mờ hoặc latent mismatch |
| GAN | Adversarial game | Một lượt generator | Bất ổn và mode collapse |
| Diffusion | Noise/score prediction | Denoise lặp lại | Chi phí sampling, artifact do conditioning |

Đây là mô tả khái quát, không phải định nghĩa đầy đủ; hệ thống hiện đại có thể kết hợp ý tưởng và objective.

## 7. Evaluation vượt ra ngoài gallery sample

Đánh giá theo mục đích: chất lượng cảm nhận, diversity, tuân thủ constraint, latency, controllability, privacy, ích lợi downstream và tỷ lệ lỗi. Automated distribution metric có thể tương quan kém với phán đoán người hoặc che lỗi hiếm. Dùng prompt/điều kiện đại diện, human review nếu phù hợp và tài liệu hóa giới hạn đã biết.

## 8. Dữ liệu, consent và safety

Training data có thể chứa nội dung riêng tư, có bản quyền, nhạy cảm hoặc gây hại. Output sinh ra có thể ghi nhớ, bắt chước, tiết lộ hay trình bày sai thông tin. Ghi provenance và quyền sử dụng data, kiểm tra memorization/nội dung không được phép, thêm provenance control nếu cần và tạo đường review bởi con người cho use case tác động cao.

## Thực hành

1. Vì sao next-token prediction là autoregressive objective?
2. Term nào của VAE khuyến khích latent mã hóa tương thích prior?
3. Mode collapse trong GAN nghĩa là gì?
4. Vì sao một sample sinh ra chất lượng cao không đủ chứng minh model đáng tin?

**Kiểm tra đáp án:** mỗi dự đoán phụ thuộc prefix trước; KL divergence term; generator chỉ tạo ít mode/thiếu diversity; một sample không cho biết diversity, độ tin cậy, tần suất lỗi hay performance qua nhiều điều kiện.
