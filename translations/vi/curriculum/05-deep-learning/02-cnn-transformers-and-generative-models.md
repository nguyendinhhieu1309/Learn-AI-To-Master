# CNN, Transformer và generative model

[Xem bài gốc bằng tiếng Anh](../../../../curriculum/05-deep-learning/02-cnn-transformers-and-generative-models.md)

## Mục tiêu học tập

Nhận biết các họ neural architecture phổ biến, hiểu giả định mỗi họ đưa vào và chọn cách evaluation phù hợp.

## 1. Convolutional neural network

Image chứa pattern cục bộ: cạnh, góc, texture và hình dạng. Convolution filter trượt qua image và dùng cùng weight tại mỗi vị trí. Weight sharing giảm số parameter và mã hóa giả định rằng feature hữu ích có thể xuất hiện ở nhiều vị trí. Layer đầu có thể phát hiện pattern đơn giản; layer sau kết hợp chúng.

Pooling hoặc strided convolution giảm spatial resolution. Cách này tăng hiệu quả nhưng có thể làm mất vị trí chính xác. Với segmentation hoặc localization, architecture thường giữ lại hay tái dựng spatial detail.

## 2. Transfer learning

Pretrained network đã học representation hữu ích từ dataset lớn. Với tác vụ mới, thay output head và training head đó, rồi có thể fine-tune một số layer trước. Dùng validation set chọn chiến lược. Kiểm tra pretraining data và evaluation set không chồng lấn theo cách làm lộ đáp án.

## 3. Attention và Transformer

Embedding ánh xạ token thành vector. Self-attention cho phép mỗi token kết hợp thông tin từ token khác. Ở dạng đơn giản, mỗi token tạo query Q, key K, value V; attention weight dựa trên độ tương tự query-key rồi dùng để kết hợp value:

$$
\mathrm{Attention}(Q,K,V)=\mathrm{softmax}(QK^T/\sqrt{d_k})V.
$$

Transformer xếp chồng attention và feed-forward layer. Cần positional information vì riêng attention không mã hóa thứ tự token. Transformer dùng được cho text, image, audio và multimodal input nhưng yêu cầu data/compute có thể lớn.

## 4. Generative model

Generative model học tạo sample mới. **Autoencoder** nén input thành representation rồi tái dựng. **GAN** training generator đối kháng discriminator; training có thể bất ổn. **Diffusion model** học đảo ngược quá trình thêm noise dần dần. Autoregressive language model lặp lại dự đoán token kế tiếp. Mỗi phương pháp có objective và failure mode khác nhau; “generative” không có nghĩa là đúng sự thật hay an toàn.

## 5. Chọn theo tác vụ và constraint

Với dataset bảng nhỏ, gradient-boosted tree có thể là điểm bắt đầu tốt hơn và rẻ hơn neural network. Với image, CNN hoặc vision transformer thường gặp. Với sequence cần ngữ cảnh dài, transformer hữu ích. Cân nhắc volume data, latency, memory, interpretability và hardware triển khai.

## Thực hành

Mô tả giả định của convolution (locality và feature dùng chung) và attention (trộn thông tin phụ thuộc nội dung). Với một tác vụ, so pretrained model có sẵn với baseline nhỏ; báo cả chất lượng và chi phí inference.

## Lỗi thường gặp

- Xem tên architecture như sự thay thế cho dataset/evaluation hợp lệ.
- Fine-tune bằng validation hoặc test example.
- So model dùng input resolution/preprocessing khác nhau mà không công bố.
- Cho rằng output sinh ra đáng tin chỉ vì trông hợp lý.
