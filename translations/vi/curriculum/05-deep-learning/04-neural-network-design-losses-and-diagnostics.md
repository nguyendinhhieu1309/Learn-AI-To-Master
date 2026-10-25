# Thiết kế neural network, loss và chẩn đoán

[Xem bài gốc bằng tiếng Anh](../../../../curriculum/05-deep-learning/04-neural-network-design-losses-and-diagnostics.md)

Architecture nên phản ánh cấu trúc input và tác vụ dự đoán. Bài này liên hệ lựa chọn thiết kế với loss, output layer và dấu hiệu training thất bại.

## 1. Computational graph và backpropagation

Network là tổ hợp các phép toán khả vi. Autodiff ghi lại phép tính rồi áp dụng chain rule ngược để tính gradient. Với layer $h=\sigma(Wx+b)$, gradient đi qua activation và phép nhân matrix để cập nhật weight cùng các layer trước.

Khi debug network, xác minh forward pass nhỏ, output shape, label mapping, loss value và gradient hữu hạn. Nếu tự cài phép toán, so một gradient với finite difference trên model rất nhỏ.

## 2. Chọn output và loss khớp target

- **Single-label multiclass:** C logit cùng cross-entropy.
- **Binary classification:** một logit cùng binary cross-entropy from logits.
- **Multi-label classification:** một sigmoid/logit độc lập mỗi label; label không loại trừ lẫn nhau.
- **Regression:** output tuyến tính cùng MSE, MAE, Huber hoặc likelihood phù hợp tác vụ.
- **Segmentation:** class logit từng pixel cùng pixel-wise loss, thường kết hợp loss nhắm overlap.

Softmax dành cho class loại trừ lẫn nhau. Áp dụng softmax từng output của multi-label task sẽ ép xác suất cộng thành 1, gây sai.

## 3. Activation và gradient

ReLU đơn giản, hiệu quả nhưng có thể tạo unit bất hoạt; GELU và SiLU là lựa chọn trơn dùng trong nhiều architecture. Sigmoid có thể bão hòa khi độ lớn lớn khiến gradient nhỏ; thường dùng ở output probability nhị phân nhưng ít dùng trong hidden layer sâu.

Initialization cần xét fan-in/fan-out và activation. Residual connection giúp giữ thông tin và luồng gradient trong network sâu. Chẩn đoán gradient biến mất/bùng nổ bằng cách log norm ở nhiều layer.

## 4. Normalization

Batch normalization dùng batch statistic khi training và running estimate khi inference. Layer normalization normalize bên trong từng example và thường gặp trong transformer. Group normalization hữu ích với vision model có batch size nhỏ. Các layer này không thể thay thế nhau ở mọi architecture; validation train/eval behavior và distributed setting.

## 5. Architecture theo cấu trúc dữ liệu

- **MLP:** vector bảng kích thước cố định hoặc kết hợp feature đã học.
- **CNN:** pattern không gian cục bộ và cấu trúc liên quan đến dịch chuyển trong image.
- **RNN:** state chuỗi với phép tính đệ quy; hữu ích trong một số bối cảnh streaming/compact.
- **Transformer:** attention trên chuỗi token/patch, cần bổ sung position information tường minh hoặc qua architecture.
- **Graph neural network:** entity có edge quan hệ.

Inductive bias có thể tăng data efficiency. Transformer không tự động tốt nhất cho tabular data nhỏ; hãy so với baseline đơn giản.

## 6. Chẩn đoán learning curve

Đọc train/validation loss cùng task metric. Nếu cả hai đều kém, điều tra optimization, data, label và model capacity. Nếu train cải thiện còn validation xấu đi, kiểm tra overfit hoặc distribution mismatch. Nếu loss cải thiện nhưng task metric không đổi, kiểm tra class imbalance, threshold, calibration, cách tính metric và label encoding.

## 7. Bài tập

1. Vì sao softmax sai với tác vụ mỗi ví dụ có thể đồng thời mang nhiều label?
2. Validation loss tăng trong khi training loss giảm. Kiểm tra nào phân biệt overfit với lỗi split?
3. Custom activation trả output hữu hạn nhưng không có gradient. Cần xem gì?
4. Vì sao group normalization có thể tốt hơn batch normalization khi GPU memory khiến batch rất nhỏ?
