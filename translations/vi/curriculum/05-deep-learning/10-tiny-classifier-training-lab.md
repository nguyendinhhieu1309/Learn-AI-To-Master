# Lab training classifier nhỏ: shape, gradient và chẩn đoán

[Xem bài gốc bằng tiếng Anh](../../../../curriculum/05-deep-learning/10-tiny-classifier-training-lab.md)

Lab này training classifier cố ý nhỏ. Mục tiêu là kiểm tra cơ chế của một lần deep-learning training trước khi tăng data, parameter hay hardware.

## Mục tiêu học tập

- Kiểm tra batch dimension và class dimension trước training.
- Dùng logits đúng với multiclass loss.
- Tách training mode khỏi evaluation mode.
- Dùng tiny-batch overfitting như một bài kiểm tra debug.
- So sánh train/validation behavior mà không tune bằng test set.

## Điều kiện tiên quyết

Hoàn thành [neural network](01-neural-networks-and-training.md), [backpropagation](05-backpropagation-worked-example.md) và [optimizer](09-optimizers-and-training-dynamics.md). Cài PyTorch phù hợp hardware theo hướng dẫn chính thức; accelerator installation khác tùy platform.

## 1. Tạo dataset nhỏ, phân tách được

Các điểm sau có hai input feature và hai class. Dataset tổng hợp này chỉ để kiểm tra training loop.

~~~python
import torch
from torch import nn

torch.manual_seed(7)

x_train = torch.tensor(
    [[-2.0, -1.0], [-1.0, -2.0], [-1.0, -1.0],
     [ 1.0,  1.0], [ 2.0,  1.0], [ 1.0,  2.0]],
    dtype=torch.float32,
)
y_train = torch.tensor([0, 0, 0, 1, 1, 1], dtype=torch.long)

x_valid = torch.tensor(
    [[-1.5, -0.5], [0.5, 1.5]],
    dtype=torch.float32,
)
y_valid = torch.tensor([0, 1], dtype=torch.long)

assert x_train.shape == (6, 2)
assert y_train.shape == (6,)
assert x_valid.shape == (2, 2)
~~~

Split này rất nhỏ, chỉ minh họa chứ không đáng tin về thống kê. Với tác vụ thật, split theo đơn vị độc lập thực tế và giữ test set cuối.

## 2. Định nghĩa model và loss

~~~python
model = nn.Sequential(
    nn.Linear(2, 8),
    nn.Tanh(),
    nn.Linear(8, 2),
)

loss_fn = nn.CrossEntropyLoss()
optimizer = torch.optim.AdamW(model.parameters(), lr=0.03, weight_decay=0.01)

logits = model(x_train)
assert logits.shape == (6, 2)
loss = loss_fn(logits, y_train)
print("initial loss:", float(loss.detach()))
~~~

Output có một logit mỗi class. Truyền raw logit vào cross-entropy; không softmax trước. Label là integer class index có shape $(N,)$. Shape label không khớp có thể âm thầm tối ưu tác vụ khác hoặc báo lỗi, vì vậy kiểm tra cả hai shape.

## 3. Training và validation

~~~python
from torch.nn.utils import clip_grad_norm_

history = []
for epoch in range(200):
    model.train()
    optimizer.zero_grad(set_to_none=True)
    train_logits = model(x_train)
    train_loss = loss_fn(train_logits, y_train)
    train_loss.backward()
    clip_grad_norm_(model.parameters(), max_norm=5.0)
    optimizer.step()

    model.eval()
    with torch.no_grad():
        valid_logits = model(x_valid)
        valid_loss = loss_fn(valid_logits, y_valid)
        valid_accuracy = (valid_logits.argmax(dim=1) == y_valid).float().mean()

    history.append(
        {
            "epoch": epoch,
            "train_loss": float(train_loss.detach()),
            "valid_loss": float(valid_loss),
            "valid_accuracy": float(valid_accuracy),
        }
    )
~~~

Với dataset lớn hơn, thay full-batch tensor bằng training data loader và cộng dồn metric qua mọi validation batch. model.train() và model.eval() đổi hành vi của layer như dropout, batch normalization. Evaluation không nên cập nhật gradient.

## 4. Thử overfit tiny batch

Training trên một số ví dụ nhỏ với capacity đủ và xác minh train loss có thể giảm rất thấp. Bài kiểm tra này xác nhận:

- input và label có thẳng hàng không;
- loss có nối với model output không;
- gradient có tới trainable parameter không;
- optimizer có thực sự update chúng không;
- model có biểu diễn được task không.

Overfit tiny batch là bài debug, không phải bằng chứng generalization. Nếu loss không giảm, kiểm tra dtype, label, logit, gradient norm, learning rate, parameter bị đóng băng trước khi thêm layer.

## 5. Kiểm tra prediction, không chỉ một con số

In hoặc vẽ:

- train và validation loss theo epoch;
- train và validation accuracy;
- class và probability dự đoán mỗi ví dụ;
- gradient norm và clipping có kích hoạt không;
- số parameter model và inference batch size.

Validation set ở đây chỉ có hai ví dụ nên một lỗi đổi accuracy 50 điểm phần trăm. Đừng diễn giải score này như ước lượng performance.

## 6. Thay đổi có kiểm soát

Mỗi lần đổi một yếu tố:

1. Giảm learning rate 10 lần.
2. Tăng hidden width.
3. Thay Tanh bằng ReLU.
4. Tăng weight decay.
5. Bỏ clipping và kiểm tra gradient norm.

Với mỗi run, ghi configuration, seed, train/validation metric và điều đã đổi. Giữ validation procedure cố định; đừng chọn setting dựa trên test set.

## Bài luyện tập kèm đáp án

1. Vì sao label tensor trong ví dụ có shape $(N,)$ thay vì $(N,2)$?
2. Vì sao không softmax trước cross-entropy ở đây?
3. Tiny-batch overfit thành công chứng minh điều gì?
4. Nó không chứng minh điều gì?
5. Vì sao validation accuracy trên hai ví dụ quá nhiễu để so model?

### Đáp án

1. Loss cần integer class index cho từng ví dụ.
2. Cross-entropy nhận logit và tự tính log-softmax; softmax trước làm đổi hành vi số và ý nghĩa input mong đợi.
3. Cho bằng chứng forward pass, label, gradient và optimizer có thể fit những ví dụ đó.
4. Không chứng minh generalization, calibration, robustness hay performance hữu ích trên population mục tiêu.
5. Mỗi ví dụ chiếm 50 điểm phần trăm nên score có sampling uncertainty cực lớn.

## Lỗi thường gặp

- Đưa probability cho loss đang yêu cầu logit.
- Evaluation khi dropout/batch normalization vẫn ở training mode.
- Báo validation score mà thiếu số lượng mẫu.
- Tăng model capacity trước khi kiểm tra data và gradient path.
- Coi seed deterministic là bảo đảm kết quả giống hệt trên mọi hardware.
