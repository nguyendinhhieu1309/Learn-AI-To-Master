# Convolution và shape của CNN

[Xem bài gốc bằng tiếng Anh](../../../../curriculum/05-deep-learning/06-convolution-and-cnn-shapes.md)

Convolutional neural network (CNN) dùng kết nối cục bộ và filter dùng chung để xử lý tín hiệu không gian. Bài này tự tính phép convolution-style 2D trên image patch nhỏ rồi suy ra shape, số parameter thường gây lỗi khi cài đặt.

## Mục tiêu học tập

- Tính phản hồi của filter 2D hợp lệ trên image patch nhỏ.
- Phân biệt filter, feature map, stride, padding và dilation.
- Tính output size cho thiết lập convolution phổ biến.
- Theo dõi channel và số parameter của convolutional layer.
- Giải thích convolution giữ lại điều gì và pooling có thể làm mất điều gì.

## 1. Convolution layer tính gì?

Trong deep-learning library, phép toán thường dùng thực ra là cross-correlation: kernel được áp dụng mà không lật. Với tọa độ image $(i,j)$ và một input channel:

$$
Y_{i,j}=\sum_{a=0}^{K_h-1}\sum_{b=0}^{K_w-1}X_{i+a,j+b}K_{a,b}+b.
$$

Filter weight được dùng chung tại mọi vị trí không gian. Filter học nhận diện cạnh sẽ phản ứng với cạnh đó ở bất cứ đâu trong image. Nhiều filter học nhiều output channel.

## 2. Ví dụ 2D có hướng dẫn

Cho:

$$
X=\begin{bmatrix}1&2&0\\0&1&3\\2&1&0\end{bmatrix},
\qquad
K=\begin{bmatrix}1&0\\0&-1\end{bmatrix}.
$$

Với stride 1 và không padding, response góc trái trên là $1(1)+2(0)+0(0)+1(-1)=0$. Trượt sang phải, xuống dưới và chéo được:

$$
Y=\begin{bmatrix}0&-1\\-2&1\end{bmatrix}.
$$

Nếu dùng bias $b=2$, cộng 2 cho mọi output. Cùng kernel weight được tái sử dụng tại cả bốn vị trí.

## 3. Kích thước output

Với một chiều không gian, input length H, kernel K, padding P, stride S và dilation D, output length:

$$
H_{\mathrm{out}}=
\left\lfloor\frac{H+2P-D(K-1)-1}{S}\right\rfloor+1.
$$

Tính tương tự độc lập cho chiều rộng. Effective kernel width là $D(K-1)+1$. Với $H=32,K=3,P=1,S=1,D=1$, output là 32. Với $S=2$, output là 16. “Same” padding là quy ước framework nhằm giữ kích thước không gian mục tiêu; kiểm tra hành vi với kernel chẵn và stride lớn hơn 1.

## 4. Channel và số parameter

Tensor input có thể lưu dạng (B,C,H,W) (channels first) hoặc (B,H,W,C) (channels last). Batch B, channel C, height H và width W là các axis khác nhau — đừng suy luận ý nghĩa chỉ từ con số.

Convolution có $C_{in}$ input channel, $C_{out}$ filter và kernel $K_h\times K_w$, số parameter học được gồm một bias mỗi filter là:

$$
C_{out}(C_{in}K_hK_w+1).
$$

Ví dụ 3 input channel, 16 filter, kernel 3×3 và bias cần $16(3\cdot3\cdot3+1)=448$ parameter. Output shape được xác định bởi batch, $C_{out}$ và hai công thức kích thước không gian ở trên.

Với nhiều input channel, mỗi filter có một kernel cho từng channel. Các response được cộng rồi thêm bias. Group convolution giới hạn channel input nào kết nối với channel output; depthwise convolution áp dụng một spatial filter mỗi input channel và thường cần pointwise layer để trộn channel.

## 5. Stride, padding, dilation và pooling

- **Stride** dịch filter nhiều pixel mỗi bước output, làm giảm resolution và compute.
- **Padding** thêm viền (thường là zero) để filter bao phủ pixel cạnh và kiểm soát output size.
- **Dilation** tạo khoảng trống giữa kernel element, tăng receptive field mà không thêm learned weight.
- **Pooling** tóm tắt vùng cục bộ, thường lấy max hoặc average. Nó giảm kích thước không gian nhưng có thể làm mất vị trí chính xác.

Downsampling tăng hiệu quả nhưng có thể có hại khi cần localization chính xác. Hệ thống detection/segmentation dùng feature pyramid, skip connection hoặc upsampling để khôi phục chi tiết không gian.

## 6. Giả định và architecture CNN

Convolution mã hóa locality và translation equivariance: dịch input thường làm feature map dịch theo. Nó không tạo translation invariance hoàn chỉnh; pooling, data augmentation hoặc aggregation phía sau có thể tăng mức chịu dịch. Cấu trúc cục bộ là inductive bias hữu ích cho image và nhiều tín hiệu sensor, nhưng không phải mọi input đều là grid tự nhiên.

Receptive field là vùng image gốc có thể ảnh hưởng đến feature. Xếp chồng layer làm nó lớn hơn. Residual block tạo đường tắt cho thông tin và gradient. Thiết kế CNN còn cần xét normalization, activation, initialization, augmentation và output stride phù hợp tác vụ.

## 7. Debug vision model

1. In chính xác input/output shape và xác định ý nghĩa từng axis.
2. Áp filter đã biết lên image nhỏ có thể kiểm tra bằng tay.
3. Xác nhận pixel scaling, thứ tự channel, label mapping và resize/crop.
4. Overfit tiny batch rồi xem overlay hoặc prediction.
5. So validation example ở cùng resolution/preprocessing như deployment.

## Bài luyện tập kèm đáp án

1. Input 28, kernel 5, padding 0, stride 1, dilation 1 thì output length là bao nhiêu? **Đáp án:** 24.
2. Có bao nhiêu weight, gồm bias, với 8 input channel và 32 filter kích thước $3\times3$? **Đáp án:** $32(8\cdot9+1)=2,336$.
3. Stride 2 convolution thường tăng spatial resolution không? **Đáp án:** không; nó downsample.
4. Vì sao dùng chung kernel ở nhiều vị trí? **Đáp án:** giảm parameter và mã hóa giả định feature có thể xuất hiện ở nhiều vị trí.
