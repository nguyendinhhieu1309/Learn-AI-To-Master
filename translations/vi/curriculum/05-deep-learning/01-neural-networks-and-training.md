# Neural network và training

[Xem bài gốc bằng tiếng Anh](../../../../curriculum/05-deep-learning/01-neural-networks-and-training.md)

## Mục tiêu học tập

Giải thích neural network như tổ hợp các hàm, theo dõi một bước training và dùng learning curve để chẩn đoán vấn đề thường gặp.

## 1. Một neuron

Neuron tính tổng có trọng số cộng bias rồi áp dụng activation:

$$
z=w^Tx+b,
\qquad a=\phi(z).
$$

Activation $\phi$ thêm tính phi tuyến. Nếu không có nonlinear activation, xếp chồng linear layer vẫn chỉ tương đương một linear transformation. Activation thường gặp:

$$
\operatorname{ReLU}(z)=\max(0,z),
\qquad \sigma(z)=\frac{1}{1+e^{-z}}.
$$

## 2. Layer và shape

Với batch X có shape (batch_size, input_dim), dense layer có hidden_dim unit thường có weight matrix shape (input_dim, hidden_dim), cho output shape (batch_size, hidden_dim). Theo dõi tường minh batch axis và feature axis. Shape mismatch thường báo axis bị đảo hoặc preprocessing sai.

Multi-layer network ghép các transformation:

$$
H^{(1)}=\phi(XW^{(1)}+b^{(1)}),
\qquad
H^{(2)}=\phi(H^{(1)}W^{(2)}+b^{(2)}),
$$

sau đó tính output score (logit). Với multiclass classification, softmax đổi logit thành class probability.

## 3. Loss và backpropagation

Loss so sánh prediction với target. Với regression thường dùng mean squared error. Với classification, cross-entropy thường áp dụng lên logit và integer class label. Backpropagation dùng chain rule tính mỗi parameter góp phần thế nào vào loss; optimizer dùng gradient để cập nhật parameter.

~~~python
for x_batch, y_batch in training_data:
    optimizer.zero_grad()
    logits = model(x_batch)
    loss = loss_function(logits, y_batch)
    loss.backward()
    optimizer.step()
~~~

API framework khác nhau nhưng ý tưởng giống nhau: xóa gradient cũ, forward pass, tính loss, backpropagate rồi update.

Với tác vụ ba class và bốn input feature, model PyTorch nhỏ có thể là:

~~~python
import torch.nn as nn

model = nn.Sequential(
    nn.Linear(4, 16),
    nn.ReLU(),
    nn.Linear(16, 3),  # raw class logits
)
loss_function = nn.CrossEntropyLoss()
~~~

CrossEntropyLoss nhận raw logits và integer class ID như 0, 1 hoặc 2; không áp dụng softmax trước loss này. Khi inference, dùng model.eval() và with torch.no_grad(): để tắt hành vi chỉ dùng khi training và không lưu gradient.

## 4. Batch, epoch và validation

**Batch** là tập con dùng cho một lần update. **Epoch** là một lượt đi qua training set. Shuffle ví dụ có thể hỗ trợ stochastic optimization nếu thứ tự không mang ý nghĩa. Không đưa validation example vào gradient update. Dùng validation curve chọn setting hoặc dừng training; giữ test set cho báo cáo cuối.

## 5. Chẩn đoán learning curve

- Train và validation loss đều cao: có thể underfit, feature/label kém hoặc optimization có vấn đề.
- Train loss thấp nhưng validation loss cao: có thể overfit hoặc train/validation khác phân phối.
- Loss thành NaN hoặc tăng vọt: kiểm tra learning rate, input không hợp lệ, normalization và numerical stability.
- Metric không cải thiện: xác minh target encoding, output layer, loss và label alignment.

Dropout, weight decay, data augmentation, early stopping và dữ liệu đại diện hơn có thể giúp chống overfit. Thêm layer không nên là cách sửa đầu tiên.

## Ví dụ có hướng dẫn

Với binary label, nếu model gán xác suất $p$ cho class đúng thì negative log-likelihood:

$$
\ell=-\ln p.
$$

Với $p=0.8$, loss $-\ln(0.8)\approx0.223$. Nếu model gán $p=0.1$, loss $-\ln(0.1)\approx2.303$: prediction sai nhưng tự tin bị phạt nặng hơn.

## Thực hành

Train network nhỏ trên tác vụ classification đồ chơi. Vẽ train/validation loss sau từng epoch. Chỉ đổi một yếu tố (ví dụ network width hoặc weight decay) rồi mô tả thay đổi. So sánh với classical baseline đơn giản.

## Lỗi thường gặp

- Áp dụng softmax hai lần khi cross-entropy loss của framework nhận logits.
- Cập nhật weight bằng validation/test batch.
- Nhầm batch size với số feature.
- Tin final training score mà không xem lỗi held-out.
