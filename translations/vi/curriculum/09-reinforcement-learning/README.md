# Reinforcement Learning

## Kết quả học tập

- Mô hình hóa tác vụ tương tác bằng state, action, reward và transition.
- Giải thích value function và trade-off exploration/exploitation.
- Huấn luyện và đánh giá agent đơn giản trong môi trường kiểm soát được.

## Kiến thức tiên quyết

Python, probability, ML cơ bản và quen với thí nghiệm lặp. Deep learning cần thiết cho deep RL.

## Lộ trình gợi ý

1. [MDP và value learning](01-mdps-and-value-learning.md)
2. [Policy learning và evaluation](02-policy-learning-and-evaluation.md)
3. [Policy gradient, actor-critic và thực hành](03-policy-gradients-actor-critic-and-practice.md)
4. [Lab thực hành thuật toán](04-algorithm-worked-lab.md)

## Bài thực hành

- Giải MDP hữu hạn nhỏ bằng tay và so với dynamic programming.
- So action ngẫu nhiên với tabular Q policy đã học.
- Chạy agent với nhiều random seed; báo cáo độ biến thiên chứ không chỉ lần chạy tốt nhất.
- Tạo ví dụ reward thiết kế kém khiến agent tìm hành vi không mong muốn.
- Tính một Q-learning update bằng tay rồi so learning curve qua các seed.

## Dự án checkpoint

Huấn luyện agent trong simulator nhỏ, so với baseline có ý nghĩa, đánh giá qua nhiều seed và giải thích reward design, failure mode cũng như lý do kết quả có thể không chuyển sang thế giới thật.
