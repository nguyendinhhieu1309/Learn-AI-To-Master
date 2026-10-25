# Tự tính attention và Transformer

[Xem bài gốc bằng tiếng Anh](../../../../curriculum/05-deep-learning/07-attention-and-transformers-by-hand.md)

Self-attention cho mỗi token tạo representation phụ thuộc ngữ cảnh bằng cách chọn và kết hợp thông tin từ token khác. Bài này theo dõi shape query/key/value, tính ví dụ attention nhỏ, giải thích mask và chi phí theo độ dài chuỗi.

## Mục tiêu học tập

- Giải thích query, key, value bằng ngôn ngữ dễ hiểu.
- Tính scaled dot-product attention cho hai token.
- Theo dõi batch, sequence, head và feature dimension.
- Phân biệt causal mask và padding mask.
- Mô tả transformer block cùng chi phí và failure mode chính.

## 1. Query, key và value

Gọi $X\in\mathbb{R}^{B\times T\times d_{model}}$ là batch B sequence có T token, mỗi token embedding trong $d_{model}$ chiều. Projection đã học tạo:

$$
Q=XW_Q,\qquad K=XW_K,\qquad V=XW_V.
$$

Trực giác: query hỏi token này cần thông tin gì; key cho biết token đang chứa thông tin nào; value mang thông tin để trộn. Đây là learned projection, không phải câu hỏi ngôn ngữ tự nhiên theo nghĩa đen.

## 2. Scaled dot-product attention

Với một head, similarity score giữa mỗi query và key:

$$
S=\frac{QK^T}{\sqrt{d_k}}.
$$

Áp dụng softmax theo các vị trí key cho từng query rồi dùng trọng số đó lấy trung bình value:

$$
A=\operatorname{softmax}(S+M),
\qquad
\operatorname{Attention}(Q,K,V)=AV.
$$

$M$ là additive mask tùy chọn; vị trí bị cấm nhận score rất âm trước softmax nên probability xấp xỉ 0. Chia cho $\sqrt{d_k}$ giúp dot product không tăng quá lớn khi key dimension tăng, tránh đẩy softmax vào vùng bão hòa.

### Ví dụ hai token có hướng dẫn

Giả sử một query $q=(1,0)$, hai key $k_1=(1,0)$ và $k_2=(0,1)$. Với $d_k=2$, logit là $[1/\sqrt2,0]\approx[0.707,0]$. Softmax cho trọng số xấp xỉ $[0.67,0.33]$. Nếu value là $v_1=(2,0)$ và $v_2=(0,4)$ thì output:

$$
0.67(2,0)+0.33(0,4)\approx(1.34,1.32).
$$

Output trộn thông tin từ hai vị trí theo similarity query-key. Trọng số phụ thuộc nội dung và thay đổi khi query/key đổi.

## 3. Multi-head attention và shape

Với h head, mỗi head dùng subspace ít chiều hơn, tính attention riêng rồi ghép output và chiếu lại. Shape thường gặp:

$$
X:(B,T,d_{model}),\quad
Q,K:(B,h,T,d_k),\quad
V:(B,h,T,d_v),
$$

$$
QK^T:(B,h,T,T),\qquad
AV:(B,h,T,d_v).
$$

Thường $d_k=d_v=d_{model}/h$. Ma trận score kích thước $T\times T$ là lý do full self-attention tốn kém với chuỗi dài. Efficient attention đổi cách tính hoặc xấp xỉ; không làm context length miễn phí.

## 4. Mask và vị trí

- **Padding mask:** ngăn attention vào padding token không phải input thật.
- **Causal mask:** ngăn token nhìn vị trí tương lai. Dùng cho next-token generation để model không nhìn đáp án trước.
- **Cross-attention mask:** có thể giới hạn vị trí encoder/source mà decoder được đọc.

Riêng attention là permutation-equivariant: sắp xếp lại token sẽ sắp xếp output tương ứng nhưng không cho model biết token nào đứng trước. Vì vậy Transformer thêm positional information qua learned position, sinusoidal encoding, relative bias hoặc phương pháp khác. Tokenization và positional representation là một phần giả định của model.

## 5. Bên trong transformer block có gì?

Block phổ biến gồm multi-head attention, feed-forward network theo từng vị trí, residual connection và normalization. Dạng đơn giản hóa:

$$
H'=\operatorname{Norm}(H+\operatorname{MHA}(H)),
\qquad
H_{next}=\operatorname{Norm}(H'+\operatorname{FFN}(H')).
$$

Thứ tự chính xác (pre-norm/post-norm), activation (ví dụ GELU/SwiGLU), normalization và masking thay đổi. Residual path hỗ trợ gradient flow; normalization ổn định scale representation. Decoder-only model dùng causal self-attention; encoder-decoder còn dùng cross-attention.

## 6. Objective training và inference

Autoregressive language model dự đoán token kế tiếp. Với token sequence $x_1,\ldots,x_T$, objective:

$$
\mathcal{L}=-\sum_{t=1}^{T}\log P(x_t\mid x_{<t}).
$$

Teacher forcing đưa token thật trước đó vào trong training. Khi generation, model điều kiện theo token chính nó đã sample/chọn, nên lỗi có thể tích lũy. Sampling strategy (greedy, temperature, top-k, nucleus) đổi diversity và risk; chúng không bổ sung tri thức thực tế.

## 7. Debug attention

- Xác minh token ID, attention mask, padding side và position index.
- Kiểm tra logit/loss trên ví dụ hai token.
- Xác nhận attention probability ở vị trí bị mask bằng 0, hàng không bị mask cộng thành 1.
- Kiểm tra training và generation dùng BOS/EOS, chat template tương thích.
- Theo dõi memory với sequence dài; attention score matrix tăng xấp xỉ $T^2$.

## Bài luyện tập kèm đáp án

1. Softmax attention chuẩn hóa theo axis nào? **Đáp án:** theo vị trí key/source cho từng query và head.
2. Causal mask chặn điều gì? **Đáp án:** key ở vị trí tương lai.
3. Vì sao chia score cho $\sqrt{d_k}$? **Đáp án:** giữ scale score ổn định khi key dimension tăng và tránh softmax quá bão hòa.
4. Attention tự mã hóa thứ tự token không? **Đáp án:** không; cần positional information.
