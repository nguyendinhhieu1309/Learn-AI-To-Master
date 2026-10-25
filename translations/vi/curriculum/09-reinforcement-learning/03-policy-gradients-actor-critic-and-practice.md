# Policy Gradient, Actor-Critic và RL trong thực tế

Value-based method học action nào tốt. Policy-gradient method học trực tiếp policy. Chúng là nền tảng của continuous control và nhiều hệ thống reinforcement learning hiện đại.

## 1. Policy objective

Stochastic policy $\pi_\theta(a\mid s)$ gán xác suất cho action. Ta muốn tối đa expected discounted return:

$$
J(\theta)=\mathbb{E}_{\tau\sim\pi_\theta}\left[\sum_{t=0}^{T}\gamma^t r_t\right]
$$

Policy-gradient theorem cho estimator tỷ lệ với:

$$
\nabla_\theta J(\theta) = \mathbb{E}\left[\nabla_\theta\log\pi_\theta(a_t\mid s_t)\,A^\pi(s_t,a_t)\right]
$$

Advantage $A^\pi$ đo action có tốt hơn hành động thường lệ của policy ở state đó hay không. Tăng log-probability của action advantage dương và giảm với action advantage âm giúp tăng expected return theo kỳ vọng.

## 2. REINFORCE và giảm variance

REINFORCE dùng return lấy mẫu làm estimate advantage nhiều nhiễu. Phương pháp đơn giản nhưng variance cao. Có thể trừ baseline $b(s)$ không phụ thuộc action được lấy mẫu mà không đổi expected gradient:

$$
\nabla_\theta J(\theta) = \mathbb{E}\left[\nabla_\theta\log\pi_\theta(a_t\mid s_t)\,(G_t-b(s_t))\right]
$$

Value function $V_\phi(s)$ đã học thường làm baseline. Chuẩn hóa advantage trong batch là lựa chọn ổn định hóa thực tế; nó đổi scale và đôi khi đổi hành vi update.

## 3. Actor-critic

Actor là policy; critic ước lượng value hoặc action value. Temporal-difference residual một bước:

$$
\delta_t = r_t + \gamma V_\phi(s_{t+1}) - V_\phi(s_t)
$$

Residual này có thể ước lượng advantage. Critic được train để dự đoán return hoặc TD target; actor update theo estimate của critic. Critic không chính xác có thể làm policy update lệch; hãy theo dõi critic loss và value prediction, đừng nghĩ actor loss nói lên toàn bộ.

Generalized Advantage Estimation kết hợp TD residual với tham số $\lambda$ để đánh đổi bias và variance:

$$
\hat{A}_t^{\mathrm{GAE}(\gamma,\lambda)} = \sum_{l=0}^{\infty}(\gamma\lambda)^l\delta_{t+l}
$$

## 4. On-policy và off-policy learning

On-policy method như PPO học từ dữ liệu do policy hiện tại/gần đây sinh ra. Off-policy method tái sử dụng experience trong replay, tăng sample efficiency nhưng cần xử lý distribution mismatch và dữ liệu cũ. Importance sampling có thể hiệu chỉnh lệch xác suất action nhưng variance có thể cao.

PPO clip tỷ lệ xác suất policy mới/cũ để hạn chế update quá lớn. Clipped objective là heuristic thực tế; nó không đảm bảo update an toàn toàn cục và không xử lý được reward misspecification.

## 5. Continuous action

Trong continuous control, policy có thể xuất tham số phân phối Gaussian, thường là mean và log standard deviation. Có thể dùng $tanh$ để giới hạn action; khi tính log probability cần tính cả phép biến đổi. Deterministic actor-critic như TD3 và SAC dành cho continuous action space; SAC tối ưu thêm entropy term nhằm khuyến khích exploration.

## 6. Reward design và exploration

Reward shaping có thể tăng tốc học nhưng tạo lối tắt khác mục tiêu tác vụ. Dùng task-level evaluation, kiểm tra hành vi và ưu tiên potential-based shaping khi giả định phù hợp. Theo dõi riêng thành phần reward và lý do episode kết thúc.

Exploration có thể dùng action noise, entropy regularization, random initial state hoặc biến thiên môi trường. Policy có thể lợi dụng bug simulator hoặc sống sót bằng cách không bao giờ thử hành vi cần thiết. Thử nhiều điều kiện và so với policy đơn giản.

## 7. Quy trình huấn luyện thực tế

1. Xác minh observation, action bound, reward scale, reset, termination và truncation flag.
2. Chạy random baseline và policy script đơn giản.
3. Cố định seed; đánh giá nhiều lần train độc lập.
4. Tách training episode khỏi evaluation episode; tắt exploration lúc đánh giá khi phù hợp.
5. Vẽ phân phối return, success rate, episode length, constraint violation và metric riêng của tác vụ.
6. Kiểm tra generalization trên initial state và environment parameter khác nhau.
7. Với robot thật, dùng mô phỏng, safety constraint, triển khai thận trọng và emergency stop.

Một seed thành công không phải bằng chứng đáng tin. Kết quả RL biến thiên đáng kể theo random seed, environment version và evaluation procedure.

## Bài tập

1. Vì sao trừ baseline chỉ phụ thuộc state có thể giảm variance mà không đổi expected policy gradient?
2. Agent reward tăng nhưng task success giảm. Cần xem gì trong reward và hành vi?
3. PPO có actor loss cải thiện nhưng value accuracy giảm mạnh. Điều này tạo rủi ro gì?
4. Vì sao báo nhiều seed và phân phối return thay vì chỉ checkpoint tốt nhất?
