# Markov Decision Process và Value Learning

## Mục tiêu học tập

Mô tả tác vụ tương tác dưới dạng MDP, tính return và hiểu value estimate hỗ trợ chọn hành động ra sao.

## 1. Vòng lặp agent-environment

Ở thời điểm $t$, agent quan sát state $s_t$, chọn action $a_t$, nhận reward $r_{t+1}$ rồi chuyển sang $s_{t+1}$. Markov decision process (MDP) gồm state, action, reward, transition dynamics và thường có discount factor $\gamma$. Markov assumption nói state chứa đủ thông tin để dự đoán phân phối state tiếp theo khi biết action.

## 2. Return và thiết kế reward

Discounted return:

$$
G_t = r_{t+1}+\gamma r_{t+2}+\gamma^2r_{t+3}+\cdots
$$

$\gamma$ gần 0 ưu tiên reward tức thời; gần 1 coi trọng reward tương lai hơn. Reward là objective do người thiết kế, không đảm bảo agent học đúng ý định. Reward kém có thể bị khai thác.

## 3. Value function và ý tưởng Bellman

$V(s)$ là return tương lai kỳ vọng từ state $s$ theo policy. $Q(s,a)$ là return kỳ vọng sau khi thực hiện action $a$ tại $s$, rồi theo policy. Bellman equation biểu diễn value thành reward tức thời cộng value state kế tiếp đã chiết khấu:

$$
Q(s,a)=\mathbb{E}[r+\gamma\max_{a'}Q(s',a')].
$$

Góc nhìn đệ quy này giúp agent học hành động nào dẫn tới state tương lai tốt.

## 4. Q-learning update

Với bảng state-action value:

$$
Q(s,a) \leftarrow Q(s,a)+\alpha[r+\gamma\max_{a'}Q(s',a')-Q(s,a)].
$$

Biểu thức trong ngoặc vuông là temporal-difference error: target trừ estimate hiện tại. $\alpha$ là learning rate. Ở terminal state, future value bằng 0.

Ví dụ $Q=2$, reward $r=1$, next-state best $Q=4$, $\gamma=0.9$, $\alpha=0.5$. Target là $1+0.9(4)=4.6$; error là $2.6$; Q mới là $2+0.5(2.6)=3.3$.

## 5. Exploration và exploitation

Luôn chọn action đang tốt nhất có thể khiến agent bỏ lỡ action tốt hơn. Epsilon-greedy chọn action ngẫu nhiên với xác suất $\epsilon$, nếu không thì chọn action đã biết tốt nhất. Giảm exploration cẩn thận; khi phù hợp, đánh giá policy cuối mà không có nhiễu exploration.

## Thực hành

Vẽ grid world nhỏ có goal và penalty. Định nghĩa state, action, terminal condition và reward. Nêu ít nhất một reward có thể khuyến khích hành vi không mong muốn. Mô tả agent biết gì ở mỗi bước.

## Lỗi thường gặp

- Quên định nghĩa terminal state hoặc thời điểm reward được phát.
- Đặt $\gamma=1$ trong tác vụ vô hạn mà không kiểm tra return có hữu hạn không.
- Dùng state giấu mất lịch sử quan trọng, vi phạm Markov assumption.
- Nghĩ reward tăng chứng minh agent hành xử đúng ý.
