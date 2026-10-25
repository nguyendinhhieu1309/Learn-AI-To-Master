# Deep Learning

[Xem trang gốc bằng tiếng Anh](../../../../curriculum/05-deep-learning/README.md)

## Kết quả học tập

- Suy ra forward pass, loss và gradient backpropagation của network nhỏ.
- Theo dõi activation, parameter và mask theo batch, token, channel và spatial shape.
- Giải thích inductive bias và chi phí tính toán của MLP, CNN, Transformer và generative model.
- Training model với split hợp lệ, optimization ổn định, record tái lập được và error analysis hữu ích.

## Điều kiện tiên quyết

Python, array/tensor, xác suất cơ bản, linear algebra, đạo hàm và workflow đánh giá ML.

## Thứ tự học gợi ý

1. [Neural network và training](01-neural-networks-and-training.md)
2. [Ví dụ backpropagation từng bước](05-backpropagation-worked-example.md)
3. [Convolution và CNN shape](06-convolution-and-cnn-shapes.md)
4. [Tự tính attention và Transformer](07-attention-and-transformers-by-hand.md)
5. [Tổng quan CNN, Transformer và generative model](02-cnn-transformers-and-generative-models.md)
6. [Nền tảng generative model](08-generative-models-foundations.md)
7. [Thực hành training deep network](03-training-deep-networks-in-practice.md)
8. [Thiết kế network, loss và chẩn đoán](04-neural-network-design-losses-and-diagnostics.md)
9. [Optimizer và động lực training](09-optimizers-and-training-dynamics.md)
10. [Lab training classifier nhỏ](10-tiny-classifier-training-lab.md)

Hãy tự làm ví dụ scalar backprop và convolution trước khi dùng framework. Sau đó kiểm tra tensor shape của framework và so một forward/gradient calculation nhỏ. Xem lại [calculus và optimization](../01-math/02-calculus-and-optimization.md) về chain rule và dùng [cheat sheet công thức](../01-math/FORMULA_CHEATSHEET.md) để ôn.

## Thực hành

- Hoàn thành [lab classifier nhỏ](10-tiny-classifier-training-lab.md), giải thích vì sao validation score không phải performance estimate.
- Làm [bộ bài tập deep learning](PROBLEM_SET_WITH_SOLUTIONS.md) để kiểm tra tensor shape, loss gradient và chẩn đoán training bằng tay.
- Cài feed-forward network nhỏ và xem tensor shape ở mỗi layer.
- So train/validation curve của setting underfit và overfit.
- Mỗi lần chỉ đổi một training setting và ghi ảnh hưởng.
- So neural network với classical baseline trên dataset nhỏ.
- Overfit một tiny batch rồi giải thích vì sao điều đó không chứng minh generalization.
- Áp filter convolution có thể tính bằng tay và xác nhận framework cho cùng feature map.
- Trực quan hóa attention weight trên sequence ngắn và giải thích causal mask chặn điều gì.
- So CNN nhỏ với pretrained model trên cùng validation data và inference setting.

## Dự án checkpoint

Training model image hoặc tabular nhỏ. Thêm baseline, lý do chọn data split, learning curve, ví dụ lỗi, configuration đã lưu và hướng dẫn tái tạo run.

## Lỗi thường gặp cần để ý

- Thêm layer trước khi kiểm tra label, leakage hoặc baseline đơn giản.
- Coi một lần chạy thành công là bằng chứng performance ổn định.
- Đổi nhiều hyperparameter cùng lúc rồi mất dấu nguyên nhân.
- Bỏ qua chi phí compute và memory khi so phương pháp.
