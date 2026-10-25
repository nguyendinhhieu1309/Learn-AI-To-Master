# Deep learning: Bộ bài tập có lời giải

[Xem bài gốc bằng tiếng Anh](../../../../curriculum/05-deep-learning/PROBLEM_SET_WITH_SOLUTIONS.md)

Hãy tự suy ra từng kết quả trước khi dùng script để kiểm tra số. Bài tập nối tensor shape, objective, gradient và quyết định training.

## 1. Affine layer

Batch có 32 ví dụ với 128 feature. Fully connected layer có 64 output cùng bias. Hãy cho output shape và số parameter.

### Lời giải

Weight có shape 128 × 64; bias có 64 giá trị. Output là 32 × 64. Số parameter: 128 × 64 + 64 = 8,256.

## 2. Softmax cross-entropy

Với logit [0,0] và target class thứ hai, tìm probability, loss và gradient theo logit.

### Lời giải

Probability là [1/2,1/2]. Loss là log 2. Gradient với one-hot target là p-y=[1/2,-1/2], vì vậy gradient descent tăng logit target tương đối với logit kia.

## 3. Binary cross-entropy with logits

Target y=1, probability dự đoán p=0.8. Tìm loss và đạo hàm theo logit z.

### Lời giải

Loss là -log(0.8), xấp xỉ 0.223. Với sigmoid cùng binary cross-entropy, $dL/dz=p-y=-0.2$.

## 4. Chain rule

Cho h=2x+1, z=3h và $L=(z-5)^2/2$. Tại x=1, tính dL/dx.

### Lời giải

Khi x=1, h=3, z=9, nên dL/dz=4, dz/dh=3 và dh/dx=2. Do đó dL/dx=4×3×2=24.

## 5. Output convolution

Image 32 × 32. Convolution dùng kernel 5 × 5, stride 1, padding 2, dilation 1. Tìm output dimension.

### Lời giải

$$
H_{out}=\left\lfloor\frac{H+2p-d(k-1)-1}{s}+1\right\rfloor.
$$

Thay số được floor((32+4-4-1)/1+1)=32. Với 16 output channel, shape channel-first là 16 × 32 × 32.

## 6. Receptive field

Hai convolution layer stride 1 dùng kernel 3 × 3, không dilation. Receptive-field width là bao nhiêu?

### Lời giải

Layer đầu nhìn vùng rộng 3; layer thứ hai mở rộng thêm 2, tổng width 5. Cũng cần theo dõi khoảng cách feature: stride làm đổi khoảng cách giữa activation lân cận.

## 7. Batch normalization và evaluation mode

Vì sao prediction có thể đổi nếu model còn ở training mode lúc evaluation?

### Lời giải

Batch normalization ở training mode dùng statistic batch hiện tại và cập nhật running estimate; evaluation mode dùng statistic đã lưu. Dropout cũng còn ngẫu nhiên nếu chưa chuyển sang evaluation mode.

## 8. Shape của attention

Query có shape [batch, heads, query_length, head_dim]; key có shape [batch, heads, key_length, head_dim]. Score có shape nào?

### Lời giải

Score có shape [batch, heads, query_length, key_length]. Causal mask chặn vị trí tương lai bị cấm. Scale dot product bằng sqrt(head_dim) kiểm soát độ lớn.

## 9. Residual shape

Activation có shape [batch,64], residual branch cho [batch,128]. Có cộng được không?

### Lời giải

Không. Chiếu skip path từ 64 lên 128 hoặc đổi residual branch thành output 64. Phép cộng đòi dimension tương thích.

## 10. Mất cân bằng class

Dataset có 990 negative và 10 positive. Classifier luôn dự đoán negative. Tính accuracy và positive recall.

### Lời giải

Accuracy 99%; positive recall 0%. Báo metric theo class và chọn threshold bằng validation data cùng chi phí deployment.

## 11. Gradient accumulation

GPU chứa được batch 8 nhưng batch hiệu dụng mong muốn là 32. Cần tích lũy bao nhiêu microbatch?

### Lời giải

Tích lũy bốn microbatch trước khi step; nếu gradient được cộng, scale mỗi loss 1/4. Batch normalization vẫn thấy batch 8 nên kết quả không giống hoàn toàn batch vật lý 32.

## 12. Learning-rate bất ổn

Loss dao động rồi thành NaN sau khi tăng learning rate. Cần kiểm tra gì trước khi đổi architecture?

### Lời giải

Kiểm tra learning rate, gradient norm, normalization, dữ liệu/loss không hữu hạn, mixed-precision scaling và loss reduction. Tìm phép toán đầu tiên tạo giá trị không hữu hạn; thử learning rate thấp hơn và clipping nếu có cơ sở.

## 13. AdamW và L2

Vì sao decoupled weight decay không nhất thiết tương đương thêm L2 penalty khi dùng adaptive optimizer?

### Lời giải

L2 penalty thêm parameter vào gradient, gradient này được adaptive moment biến đổi. Decoupled decay shrink parameter riêng khỏi adaptive update nên bước kết quả khác.

## 14. Kiểm tra tiny-batch overfit

Network không giảm được loss gần 0 trên 12 ví dụ cố định khi tắt augmentation. Nêu bug có khả năng.

### Lời giải

Kiểm tra input/label alignment, target encoding, output/loss tương thích, gradient flow, optimizer đã đăng ký parameter chưa, learning rate, accidental detach/no-grad, train/eval mode và transform. Xem prediction/gradient trước khi scale up.

## 15. Lặp lại việc chọn theo test

Bạn thử 40 architecture rồi báo test accuracy tốt nhất. Vì sao có thể thiên lệch dù test label không cập nhật gradient?

### Lời giải

Việc chọn lặp dùng test result làm feedback. Random variation khiến model thắng trông tốt bất thường. Tune trên train/validation và dành test cho ước lượng cuối có giới hạn.

## Checklist ôn tập

- Bạn có tự tính tensor shape và số parameter được không?
- Bạn có suy ra output-layer gradient cho loss thường gặp không?
- Train/eval mode có dùng normalization/dropout như dự định?
- Bạn có giải thích được optimizer update không?
- Bạn có debug tiny-batch overfit trước khi scale training?
- Model selection có được tách khỏi báo cáo test cuối?
