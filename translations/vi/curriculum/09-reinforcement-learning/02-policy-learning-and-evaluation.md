# Policy Learning và Evaluation

## Mục tiêu học tập

Hiểu khác biệt giữa value-based và policy-based method, đánh giá công bằng quá trình huấn luyện ngẫu nhiên và nhận biết giới hạn sim-to-real.

## 1. Từ value tới policy

**Policy** $\pi(a\mid s)$ mô tả action cần chọn trong một state. Value-based method học $Q(s,a)$ rồi chọn action có value cao. Policy-based method tối ưu trực tiếp policy; hữu ích với action ngẫu nhiên hoặc liên tục. Actor-critic kết hợp policy (actor) với value estimate (critic).

## 2. Trực giác policy gradient

Policy gradient tăng xác suất action dẫn đến outcome tốt hơn kỳ vọng và giảm xác suất action kém hơn. Baseline như state value giảm phương sai gradient mà không đổi hướng lý tưởng. Huấn luyện có nhiễu vì dữ liệu phụ thuộc policy và environment hiện tại.

Deep Q-network dùng function approximation cho state space lớn, cùng kỹ thuật như replay buffer và target network để ổn định học. Chúng giúp ích nhưng không làm training tất định hoặc luôn ổn định.

## 3. Đánh giá nhiều lần chạy

Dùng nhiều random seed và báo cáo phân phối (ví dụ median cùng độ phân tán), không chỉ lần tốt nhất. So với random, heuristic hoặc policy lập trình đơn giản. Vẽ reward và success rate; xem trajectory, vi phạm constraint và failure mode. Cố định version và setting môi trường.

## 4. Reward hacking và safety

Agent tối ưu reward nó nhận, kể cả loophole vô tình. Kiểm tra state bất thường, hệ quả trễ và điều kiện biên. Khi cần safety, dùng constraint hoặc safety shield; đừng chỉ dựa vào reward penalty. Offline RL học từ action đã log và phải xử lý distribution mismatch: offline dataset có thể không chứa action an toàn/thành công cho mọi tình huống.

## 5. Sim-to-real

Policy trong mô phỏng có thể thất bại trên robot thật vì physics, sensor, timing, vật liệu và nhiễu khác nhau. Domain randomization cho policy tiếp xúc nhiều biến thiên nhưng không chứng minh safety. Xác thực theo từng bước: offline, simulation, test hardware có kiểm soát, rồi mới vận hành rộng hơn cùng giới hạn và emergency stop.

## Thực hành

Huấn luyện policy trong simulator nhỏ với ít nhất năm seed. So với baseline. Báo trung bình và biến thiên, đưa episode thành công/thất bại, thử một điều kiện môi trường đã đổi. Giải thích còn cần bằng chứng nào trước khi triển khai thật.

## Lỗi thường gặp

- Chỉ báo cáo lần chạy may mắn nhất.
- Nhầm reward với mục tiêu thật hoặc safety requirement.
- Cho rằng simulation thành công sẽ chuyển nguyên vẹn sang phần cứng.
- Tune trên benchmark cố định nhưng bỏ qua variation và constraint.
