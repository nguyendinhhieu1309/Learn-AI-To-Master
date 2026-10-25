# Backpropagation: Ví dụ giải từng bước

[Xem bài gốc bằng tiếng Anh](../../../../curriculum/05-deep-learning/05-backpropagation-worked-example.md)

Backpropagation áp dụng chain rule hiệu quả trên computation graph của network. Bài này tính forward pass, loss, gradient và parameter update hoàn chỉnh cho network rất nhỏ, sau đó liên hệ với automatic differentiation.

## Mục tiêu học tập

- Theo dõi shape của activation, weight và gradient qua network.
- Tự tính forward pass và loss.
- Dùng chain rule truyền error signal ngược lại.
- Giải thích vì sao một giá trị trung gian được chia sẻ tích lũy gradient từ nhiều nhánh.
- Nhận biết giới hạn của gradient suy ra bằng tay và lúc nào nên dùng autodiff.

## 1. Network và ký hiệu

Dùng một ví dụ có hai input feature, một ReLU hidden unit và một linear output:

$$
z_h=u_1x_1+u_2x_2+b_h,
\qquad h=\operatorname{ReLU}(z_h),
\qquad \hat y=vh+b_o.
$$

Parameter là $u_1,u_2,b_h,v,b_o$. Với batch B hàng, X có shape (B,2); hidden weight (2,1) tạo activation (B,1), output weight cho kết quả (B,1). Giữ batch axis để tránh trộn nhầm ví dụ.

## 2. Tự tính forward pass

Chọn $x_1=1,x_2=2$, target $y=1$ và parameter:

$$
u_1=0.1,\;u_2=0.2,\;b_h=0.1,\;v=0.5,\;b_o=0.
$$

Hidden pre-activation và activation:

$$
z_h=0.1(1)+0.2(2)+0.1=0.6,
\qquad h=\operatorname{ReLU}(0.6)=0.6.
$$

Output và half-squared loss:

$$
\hat y=0.5(0.6)+0=0.3,
\qquad L=\frac12(\hat y-y)^2=\frac12(-0.7)^2=0.245.
$$

## 3. Backward pass bằng chain rule

Bắt đầu từ loss và đi ngược. Đặt $\delta_o=\partial L/\partial\hat y$:

$$
\delta_o=\hat y-y=-0.7,
\qquad
\frac{\partial L}{\partial v}=\delta_o h=-0.42,
\qquad
\frac{\partial L}{\partial b_o}=\delta_o=-0.7.
$$

Output phụ thuộc hidden activation qua v, do đó:

$$
\frac{\partial L}{\partial h}=\delta_o v=-0.35.
$$

Vì $z_h=0.6>0$, đạo hàm ReLU tại điểm này bằng 1, nên $\delta_h=\partial L/\partial z_h=-0.35$. Vì vậy:

$$
\frac{\partial L}{\partial u_1}=\delta_hx_1=-0.35,
\quad
\frac{\partial L}{\partial u_2}=\delta_hx_2=-0.70,
\quad
\frac{\partial L}{\partial b_h}=\delta_h=-0.35.
$$

Hidden error signal nhân với từng input đã đóng góp vào pre-activation. Với nhiều unit hoặc đường truyền, gradient của parameter là tổng đóng góp từ mọi đường đi.

## 4. Thực hiện một bước gradient descent

Với learning rate $\eta=0.1$, trừ gradient khỏi từng parameter:

$$
u_1=0.135,\quad u_2=0.270,\quad b_h=0.135,
\quad v=0.542,\quad b_o=0.070.
$$

Hidden value mới là $0.135+0.270(2)+0.135=0.81$, prediction mới là $0.542(0.81)+0.070\approx0.509$. Loss giảm còn khoảng 0.120. Điều đó xác nhận bước cập nhật đã giảm loss của ví dụ này; riêng nó không nói lên hiệu năng trên ví dụ chưa thấy.

## 5. Công thức tổng quát cho layer

Với batch theo hàng, layer l tính:

$$
Z^{(l)}=A^{(l-1)}W^{(l)}+b^{(l)},
\qquad A^{(l)}=\phi(Z^{(l)}).
$$

Nếu batch size B, input width $d_{l-1}$ và output width $d_l$ thì:

$$
A^{(l-1)}:(B,d_{l-1}),\quad
W^{(l)}:(d_{l-1},d_l),\quad
Z^{(l)}:(B,d_l).
$$

Cho gradient theo activation kế tiếp, backprop nhân với đạo hàm activation rồi nhân với weight matrix transpose phù hợp. Với affine layer cuối, loss lấy trung bình theo batch và error matrix $\Delta^{(l)}=\partial L/\partial Z^{(l)}$:

$$
\frac{\partial L}{\partial W^{(l)}}=(A^{(l-1)})^T\Delta^{(l)},
\qquad
\frac{\partial L}{\partial b^{(l)}}=\sum_{i=1}^{B}\Delta_i^{(l)}.
$$

Phép reduction chính xác theo batch phụ thuộc loss được cộng hay lấy trung bình. Vì vậy loss reduction và gradient scale cần ghi trong experiment note.

## 6. Automatic differentiation và kiểm tra gradient

Framework dựng computation graph và tự áp dụng chain rule. Autodiff không biết model/target của bạn có đúng về khái niệm không; nó tính đạo hàm chính xác theo chương trình đã viết. Kiểm tra shape, loss value và gradient hữu hạn trước run lớn.

Với custom operation, so một analytic gradient với central finite difference trên input double precision nhỏ:

$$
g_j^{\mathrm{numeric}}=
\frac{L(\theta+\epsilon e_j)-L(\theta-\epsilon e_j)}{2\epsilon}.
$$

Tránh kiểm tra đúng tại điểm không khả vi như ReLU input bằng 0. Finite difference là công cụ debug, không phải phương pháp training có thể mở rộng.

## 7. Vấn đề gradient thường gặp

- **Vanishing gradient:** đạo hàm nhỏ hơn một lặp lại làm tín hiệu ở layer sớm co dần.
- **Exploding gradient:** thừa số lớn hơn một lặp lại làm độ lớn gradient tăng vọt.
- **ReLU unit chết:** activation luôn âm nên gradient qua unit bằng 0.
- **Objective/label sai:** gradient có thể đúng về toán học nhưng đang tối ưu target không chủ ý.
- **Reduction hoặc mask sai:** gradient bao gồm padding hoặc ví dụ đáng lẽ bị loại.

Kiểm tra gradient norm theo layer, output/target shape và khả năng overfit tiny batch. Clipping giới hạn gradient cực đoan nhưng không sửa data/loss definition hỏng.

## Bài luyện tập kèm đáp án

1. Nếu output error $\delta_o=-0.7$ và activation $h=0.6$, $\partial L/\partial v$ là bao nhiêu? **Đáp án:** $-0.42$.
2. Nếu ReLU pre-activation âm, gradient qua unit đó là gì? **Đáp án:** 0, ngoài điểm gấp khúc bằng 0.
3. Vì sao bias gradient được cộng qua batch? **Đáp án:** bias dùng chung đóng góp vào pre-activation của mọi ví dụ nên cộng gradient từng ví dụ, rồi chia nếu loss lấy trung bình.
4. Autodiff có phát hiện label bị ghép nhầm input không? **Đáp án:** không; nó chỉ đạo hàm chương trình được đưa vào.
