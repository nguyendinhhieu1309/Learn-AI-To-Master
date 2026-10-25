# Hệ thống gợi ý (Recommendation Systems)

[Xem bài gốc bằng tiếng Anh](../../../../curriculum/04-machine-learning/13-recommendation-systems.md)

Recommendation system xếp hạng item cho user, chẳng hạn sản phẩm, video hay tài liệu. Thách thức không chỉ là dự đoán sở thích: feedback thưa, exposure bị thiên lệch, candidate tốt phụ thuộc ràng buộc sản phẩm và ngữ cảnh người dùng.

## Mục tiêu học tập

- Phân biệt explicit rating với implicit interaction signal.
- Giải thích content-based, neighborhood và latent-factor method.
- Suy ra prediction của matrix factorization và objective có regularization.
- Chọn ranking metric và split đánh giá an toàn với leakage.
- Nhận biết cold-start, exposure bias và vấn đề thử nghiệm online.

## 1. Training data mang ý nghĩa gì?

Explicit rating (ví dụ điểm 1–5) là tín hiệu sở thích trực tiếp nhưng thường thưa. Implicit interaction (click, view, purchase, watch time) phong phú nhưng nhập nhằng: không click có thể là “không thích”, “chưa từng được hiển thị”, “đã thấy nơi khác” hoặc “không để ý”. Positive/negative example phải phản ánh item đã được exposure ra sao.

Định nghĩa objective trước khi fit model: dự đoán rating, xếp hạng item liên quan, tăng engagement đạt chuẩn hay thỏa constraint như diversity. Các objective này không thể hoán đổi cho nhau.

## 2. Các họ model chính

- **Popularity:** gợi ý toàn cục hoặc theo segment; là baseline mạnh và giải quyết một số trường hợp cold-start.
- **Content-based:** biểu diễn user/item bằng metadata hoặc feature text/vision; xếp hạng item phù hợp sở thích quan sát được. Có thể gợi ý item mới nếu có feature, nhưng dễ quá chuyên biệt.
- **Neighborhood collaborative filtering:** gợi ý item được user tương tự thích hoặc user tương tự với người đang xét. Similarity có thể tính từ co-rating hay interaction.
- **Latent-factor model:** học vector gọn của user/item từ pattern tương tác.
- **Hybrid model:** kết hợp collaborative signal, item feature, context và constraint kinh doanh.

## 3. Matrix factorization

Gọi $R_{ui}$ là rating quan sát từ user $u$ cho item $i$. Xấp xỉ ma trận chỉ có một phần giá trị quan sát bằng user vector $p_u$ và item vector $q_i$:

$$
\hat R_{ui}=\mu+b_u+b_i+p_u^Tq_i,
$$

trong đó $\mu$ là global mean, $b_u,b_i$ là bias của user/item. Với cặp rating quan sát $\Omega$, fit factor bằng cách tối thiểu hóa:

$$
J=\sum_{(u,i)\in\Omega}(R_{ui}-\hat R_{ui})^2
+\lambda\left(\sum_u\|p_u\|_2^2+\sum_i\|q_i\|_2^2\right).
$$

Chỉ entry quan sát mới đóng góp vào rating loss; rating thiếu không tự động là zero. Cần regularization vì nhiều cặp factor có thể giải thích observation thưa. Latent dimension biểu diễn pattern lặp lại nhưng không bảo đảm có nghĩa trực quan.

Với implicit feedback, dùng ranking loss hoặc objective có trọng số confidence thay vì diễn giải mọi interaction vắng mặt là dislike. Pairwise objective như Bayesian Personalized Ranking training để score item positive đã quan sát cao hơn negative được lấy mẫu:

$$
\ell=-\log\sigma(s(u,i^+)-s(u,i^-)).
$$

Quy trình lấy negative sample ảnh hưởng đáng kể đến điều model học.

## 4. Đánh giá offline

Dùng ranking metric như Recall@K, Precision@K, NDCG@K, MAP hoặc mean reciprocal rank khi sản phẩm hiển thị danh sách xếp hạng. Mỗi metric trả lời câu hỏi khác: NDCG giảm trọng số vị trí thấp; Recall@K đo số item liên quan trong top K; precision đo tỷ lệ item đã hiển thị là relevant theo label được chọn.

Chia theo thời gian nếu sản phẩm gợi ý interaction tương lai. Với mỗi user, đảm bảo test interaction xảy ra sau train interaction; tránh để metadata item tương lai hay user aggregate leak ngược về quá khứ. Random row split có thể đưa event tương lai của user vào train và event cũ vào test, tạo ước lượng không thực tế.

Offline test chỉ đo performance theo exposure policy đã log. Model có thể trông yếu với item user chưa từng thấy; click không tự động chứng minh hài lòng. Online experiment cần guardrail cho latency, diversity, novelty, complaint và outcome ngoài ý muốn — không chỉ click.

## 5. Cold start và ràng buộc production

- User mới ít interaction history; dùng onboarding signal, context hoặc popularity baseline an toàn.
- Item mới cần content feature hoặc exploration để thu feedback.
- User/item thưa interaction có factor estimate bất định.
- Filtering rule, inventory, policy và freshness constraint cần được áp dụng và log tường minh.
- Candidate retrieval và ranking thường là hai giai đoạn; retrieval phải đưa đủ candidate tốt để ranker chọn.

Theo dõi chính xác event exposure, recommendation và feedback. Không có exposure log thì khó biết item bị bỏ qua hay chưa hề được trình bày.

## Thực hành

1. Vì sao cặp user-item chưa quan sát không nhất thiết là negative rating?
2. Split nào tốt hơn để dự đoán engagement tháng sau: random row hay theo thời gian? Vì sao?
3. Item mới không có interaction nhưng có metadata tốt. Họ model nào có thể xếp hạng trước khi có collaborative signal?
4. Nêu một lý do khiến offline Recall@10 khác kết quả online.

**Kiểm tra đáp án:** item có thể chưa từng được exposure; split theo thời gian phù hợp dự đoán tương lai; content-based hoặc hybrid; exposure bias, metric khác mục tiêu sản phẩm, feedback loop hoặc hành vi user thay đổi.
