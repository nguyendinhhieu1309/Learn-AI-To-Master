# Lab Reinforcement Learning: Bandit, Q-Learning và Policy Gradient

Reinforcement learning (RL) dùng cho quyết định tuần tự, nơi action ảnh hưởng state và reward tương lai. Agent không nhận nhãn action đúng cho mọi state; nó học qua tương tác, vì thế evaluation và exploration an toàn rất quan trọng.

## Mục tiêu học tập

- Định nghĩa tác vụ thành Markov decision process.
- Giải thích return, value function và Bellman equation.
- Tính tabular Q-learning update.
- Phân biệt value-based learning với policy-gradient method.
- Giải thích replay buffer, target network và PPO clipping.
- Đánh giá agent qua nhiều seed và chẩn đoán reward hacking.

## Kiến thức tiên quyết

Học [MDP và value learning](01-mdps-and-value-learning.md), [policy learning và evaluation](02-policy-learning-and-evaluation.md), [policy gradient](03-policy-gradients-actor-critic-and-practice.md).

## 1. Bắt đầu với multi-armed bandit

Bandit có action với phân phối reward chưa biết nhưng không có chuyển state. Ở bước $t$, learner chọn $a_t$ rồi quan sát $r_t$.

Regret so reward tích lũy với kỳ vọng nếu luôn chọn action tốt nhất:

$$
R_T=T\mu^*-\sum_{t=1}^{T}\mathbb{E}[r_t],
\qquad
\mu^*=\max_a\mathbb{E}[r\mid a].
$$

Chiến lược explore-exploit đôi khi phải thử action chưa chắc chắn để biết nó có tốt hơn không. Chỉ exploitation có thể mắc kẹt với action trông tốt do kết quả ngẫu nhiên ban đầu.

Epsilon-greedy chọn action ngẫu nhiên với xác suất $\epsilon$, nếu không chọn action có estimated value hiện tại cao nhất. Cách này đơn giản, nhưng exploration ngẫu nhiên có thể lãng phí khi action nào đó rõ ràng nguy hiểm hoặc tốn kém.

## 2. Markov decision process

MDP thường viết $(\mathcal S,\mathcal A,P,R,\gamma)$:

- $\mathcal S$: state;
- $\mathcal A$: action;
- $P(s'\mid s,a)$: xác suất transition;
- $R(s,a,s')$: reward;
- $\gamma\in[0,1)$: discount factor.

Với policy $\pi$, return từ thời điểm $t$:

$$
G_t=\sum_{k=0}^{\infty}\gamma^k r_{t+k+1}.
$$

State-value function là $V^\pi(s)=\mathbb{E}_\pi[G_t\mid s_t=s]$. Action-value function là $Q^\pi(s,a)=\mathbb{E}_\pi[G_t\mid s_t=s,a_t=a]$.

Bellman optimality relation cho action value:

$$
Q^*(s,a)=\mathbb{E}\left[r+\gamma\max_{a'}Q^*(s',a')\mid s,a\right].
$$

Phép max xuất hiện vì phương trình mô tả policy tương lai tối ưu, không nhất thiết là policy hành vi đã thu thập transition.

## 3. Tabular Q-learning update

Q-learning cập nhật value của state-action pair đã quan sát:

$$
Q(s,a)\leftarrow Q(s,a)+\alpha\left[
r+\gamma\max_{a'}Q(s',a')-Q(s,a)
\right].
$$

Biểu thức trong ngoặc là temporal-difference error. Với terminal transition, đặt next value dùng để bootstrap bằng 0.

### Tính một update

Giả sử $Q(s,a)=2$, reward $r=1$, $\gamma=0.9$, action value lớn nhất ở state kế tiếp là 4 và $\alpha=0.5$:

$$
\delta=1+0.9(4)-2=2.6,
\qquad
Q_{\mathrm{new}}(s,a)=2+0.5(2.6)=3.3.
$$

Một update đưa estimate gần sampled target 4.6 hơn. Nó không chứng minh estimate đúng; có thể cần nhiều lần thăm và exploration thích hợp.

## 4. Từ bảng tới neural value function

Khi state space lớn, neural network có thể xấp xỉ $Q(s,a;\theta)$. Cấu hình Deep Q-Network (DQN) thường dùng:

- replay buffer tái sử dụng transition ít tương quan hơn;
- target network riêng để ổn định bootstrapped target;
- exploration policy như epsilon-greedy;
- loss giữa action value dự đoán và target.

Target network được cập nhật định kỳ hoặc chậm. Replay buffer làm thay đổi phân phối training; nó không tạo dữ liệu độc lập nếu nhiều transition đến từ cùng trajectory. Reward sai hoặc policy thu thập dữ liệu không an toàn vẫn là vấn đề.

## 5. Policy gradient

Thay vì ước lượng value của từng action, policy-gradient method parameterize trực tiếp $\pi_\theta(a\mid s)$. Objective là expected return:

$$
J(\theta)=\mathbb{E}_{\tau\sim\pi_\theta}[G(\tau)].
$$

Score-function identity:

$$
\nabla_\theta J(\theta)
=\mathbb{E}\left[
\nabla_\theta\log\pi_\theta(a_t\mid s_t)\,\hat A_t
\right],
$$

trong đó $\hat A_t$ ước lượng action tốt hơn baseline kỳ vọng bao nhiêu. Value function học được có thể giảm variance, nhưng không xóa variance hoặc đảm bảo mọi update hữu hạn đều cải thiện policy.

## 6. PPO và clipped update

Proximal Policy Optimization (PPO) so xác suất policy mới với policy hành vi:

$$
r_t(\theta)=\frac{\pi_\theta(a_t\mid s_t)}
{\pi_{\theta_{\mathrm{old}}}(a_t\mid s_t)}.
$$

Clipped surrogate objective:

$$
L^{\mathrm{clip}}(\theta)=
\mathbb{E}_t\left[
\min\left(
r_t(\theta)\hat A_t,\,
\operatorname{clip}(r_t(\theta),1-\epsilon,1+\epsilon)\hat A_t
\right)\right].
$$

Clipping ngăn tỷ lệ xác suất đổi quá lớn trong objective này. Nó không phải hard trust-region guarantee, không ngăn mọi policy change gây hại và không thay thế evaluation.

## 7. Evaluation protocol

Kết quả RL có thể thay đổi đáng kể giữa random seed và độ ngẫu nhiên môi trường. Báo cáo:

- environment version và episode horizon;
- định nghĩa reward và điều kiện kết thúc;
- số lần tương tác khi train và update budget;
- reward curve cùng độ bất định qua các seed;
- success rate và safety metric riêng của tác vụ;
- random hoặc scripted baseline;
- đánh giá trên episode riêng, không update model;
- độ nhạy với initial state hoặc dynamics đã thay đổi.

Không chỉ báo seed tốt nhất hoặc episode cuối. Moving average có thể che instability; hãy thể hiện raw range hoặc confidence interval.

## 8. Reward hacking và constraint

Agent tối ưu reward nó nhận, không phải mục tiêu không chính thức trong mô tả dự án. Nếu reward bỏ sót constraint, agent có thể khai thác. Phân biệt:

- objective term biểu thị trade-off mong muốn;
- hard safety constraint tuyệt đối không được vi phạm;
- evaluation metric không dùng lúc training;
- human review và emergency stop behavior.

Với robotics hoặc hệ vật lý, bắt đầu trong simulator, giới hạn action, thử failure case và yêu cầu safe fallback. Reward mô phỏng cao không đủ chứng minh an toàn thế giới thật.

## Bài tập có đáp án kiểm tra

1. Trong Q-learning update, next state kết thúc episode dùng value nào?
2. Vì sao Q-learning lấy max next action dù behavior policy có exploration?
3. Replay buffer giảm vấn đề nào, và không giải quyết vấn đề nào?
4. Vì sao cần nhiều training seed?
5. Nêu ví dụ reward hacking trong tác vụ điều hướng.

### Đáp án

1. Future value bằng 0, nên target chỉ còn reward quan sát được.
2. Q-learning là off-policy: target đánh giá next action tham lam dù behavior đôi khi exploration.
3. Buffer giảm tương quan thời gian mạnh giữa các update và cho phép tái sử dụng transition. Nó không sửa dữ liệu lệch, reward sai hoặc coverage kém.
4. Tối ưu ngẫu nhiên và độ ngẫu nhiên môi trường khiến hiệu năng biến thiên; nhiều seed cho biết độ ổn định.
5. Reward chỉ khuyến khích tới tọa độ có thể bị lợi dụng bằng cách đi qua vùng nguy hiểm hoặc va chạm nếu collision cost thiếu/quá thấp.

## Mini-project

Cài môi trường tabular nhỏ với ít nhất bốn state và hai action. Định nghĩa terminal condition, so random behavior với Q-learning, báo learning curve qua ít nhất mười seed, thử một thay đổi về transition hoặc reward. Giải thích kết quả nào được đảm bảo bởi thiết kế và kết quả nào chỉ quan sát thực nghiệm.

## Lỗi thường gặp

- Coi reward là mô tả rõ ràng tuyệt đối của mục tiêu thực.
- So policy dùng interaction budget khác nhau.
- Đánh giá giữa lúc training còn exploration rồi gọi đó là deployment performance.
- Chỉ báo một lần chạy, không có độ biến thiên seed.
- Nghĩ clipping hoặc target network chứng minh safety.
