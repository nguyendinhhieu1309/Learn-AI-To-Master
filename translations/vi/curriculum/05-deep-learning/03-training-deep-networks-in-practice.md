# Thực hành training deep network

[Xem bài gốc bằng tiếng Anh](../../../../curriculum/05-deep-learning/03-training-deep-networks-in-practice.md)

Bài này biến định nghĩa về layer/loss thành workflow training có kỷ luật. Nội dung áp dụng cho neural network xử lý image, text, audio và dữ liệu dạng bảng.

## 1. Neural network như hàm có parameter

Model ánh xạ input $x$ thành prediction $f_\theta(x)$. Training tối thiểu hóa objective thực nghiệm trên ví dụ, thường kèm regularization:

$$
J(\theta) = \frac{1}{N}\sum_{i=1}^{N} \ell\big(f_\theta(x_i), y_i\big) + \lambda\,\Omega(\theta)
$$

Ở đây $\ell$ đo lỗi prediction, $\Omega$ phạt cấu hình parameter không mong muốn và $\lambda$ điều khiển đánh đổi. Với classification, truyền logit vào implementation cross-entropy ổn định số; không softmax hai lần.

## 2. Training loop

Mỗi iteration chạy forward pass, tính loss, tính gradient, cập nhật parameter và reset gradient. Minibatch ước lượng gradient toàn dataset với ít compute hơn mỗi lần update.

~~~python
for x, y in train_loader:
    optimizer.zero_grad(set_to_none=True)
    logits = model(x)
    loss = criterion(logits, y)
    loss.backward()
    torch.nn.utils.clip_grad_norm_(model.parameters(), max_norm=1.0)
    optimizer.step()
~~~

Evaluation ở inference mode; theo dõi validation metric riêng và lưu checkpoint tốt nhất theo metric đã chọn trước training. Log optimizer, learning rate, batch size, seed, data version, model configuration và checkpoint identity.

## 3. Optimizer và learning rate

Stochastic gradient descent cập nhật parameter bằng minibatch gradient nhiễu. Momentum tích lũy hướng đã làm trơn. Adam thích nghi step size từng parameter bằng moving average của gradient và bình phương gradient. AdamW tách weight decay khỏi gradient update của Adam, thường là mặc định cho transformer.

Learning rate thường là setting quan trọng nhất của optimization. Rate quá cao gây divergence/dao động; quá thấp làm tiến độ chậm hoặc kẹt ở vùng nghiệm kém. Với một số bài training transformer hay batch lớn, dùng warmup rồi decay bằng schedule hợp run. Learning-rate range test ngắn giúp tìm bậc độ lớn an toàn.

Nếu loss thành NaN hoặc Inf, kiểm tra giá trị data, logit, mixed precision, learning rate và gradient norm. Gradient clipping giới hạn update cực đoan nhưng không sửa label sai hay phép toán không hợp lệ.

## 4. Batch size và gradient accumulation

Batch lớn tận dụng hardware tốt hơn nhưng tốn memory và cho ít lần cập nhật parameter hơn mỗi epoch. Gradient accumulation cộng gradient qua microbatch nhỏ để xấp xỉ batch lớn, trong khi batch-normalization vẫn phụ thuộc từng microbatch. Ghi effective batch size và số optimizer update trong experiment record.

## 5. Regularization và generalization

- Weight decay hạn chế weight quá lớn.
- Dropout tắt ngẫu nhiên activation trong training.
- Data augmentation tạo biến thể input hợp lý.
- Early stopping chọn checkpoint trước khi validation performance giảm.
- Label smoothing làm mềm one-hot target trong một số bài classification.

Regularization cần khớp bài toán. Augmentation quá mạnh tạo sample phi thực tế; dropout không tự động hữu ích ở mọi nơi. Validation gap cũng có thể do leakage, split duplicate hoặc distribution shift.

## 6. Khởi tạo, normalization và activation

Khởi tạo kém có thể làm activation/gradient biến mất hoặc bùng nổ. Xavier-style initialization thường ghép với activation giống tanh; He initialization thường dùng với ReLU-family. Normalization layer ổn định một số architecture nhưng train/eval mode quan trọng. Batch normalization dùng batch statistic khi training và running statistic khi inference; quên eval mode có thể đổi output.

Residual connection tạo đường đi ngắn hơn cho gradient và hỗ trợ network sâu. Nếu training bất ổn, theo dõi activation và gradient scale.

## 7. Mixed precision và reproducibility

Mixed precision giảm memory và tăng throughput trên hardware hỗ trợ. Dùng autocast và gradient scaling của framework khi phù hợp; phép reduction nhạy số có thể vẫn cần precision cao. So validation quality với baseline full precision.

Đặt seed để có thể lặp lại, nhưng đừng hứa bitwise-identical giữa hardware, kernel hay cấu hình phân tán khác nhau. Báo biến động qua nhiều seed nếu nó ảnh hưởng kết luận.

## 8. Debug trước khi mở rộng

Thực hiện kiểm tra theo thứ tự:

1. Xác minh tensor shape, label range, class mapping và preprocessing.
2. Overfit một batch nhỏ. Nếu training loss không giảm được rất thấp, kiểm tra pipeline, loss, gradient và optimizer.
3. So với baseline đơn giản và sanity check bằng random label.
4. Vẽ train/validation loss và task metric mỗi epoch.
5. Kiểm tra prediction thực tế, không chỉ score tổng hợp.
6. Sau đó mới tăng model size, data volume hay distributed complexity.

Tiny-batch test là cách mạnh để kiểm tra implementation, không phải bằng chứng generalization.

## 9. Compute và theo dõi thí nghiệm

Ghi chính xác dataset snapshot và split ID, revision source code, dependency, hardware, random seed, thời gian training và checkpoint. Theo dõi learning rate, gradient norm, throughput, GPU memory, loss và validation metric. Model artifact thiếu preprocessing và label mapping là artifact chưa đầy đủ.

## 10. Bài tập

1. Training loss phẳng ngay từ bước đầu. Kể bốn điều cần kiểm tra trước khi đổi architecture.
2. Training accuracy tăng trong khi validation loss xấu đi. Cần kiểm tra gì và chọn checkpoint triển khai ra sao?
3. Mixed precision nhanh gấp đôi nhưng đôi khi tạo NaN. Nên thêm instrumentation và biện pháp bảo vệ nào?
4. Vì sao gradient accumulation có thể không khớp hoàn toàn một batch vật lý lớn?
