# Đánh giá Tác động, Fairness, Privacy và Oversight

## Mục tiêu học tập

Xác định ai có thể chịu ảnh hưởng bởi AI system, tìm rủi ro trong dữ liệu/cách sử dụng và viết safeguard thực tế có thể theo dõi.

## 1. Bắt đầu từ vấn đề, không phải model

Hỏi có nên tự động hóa tác vụ này không, lựa chọn nào khác tồn tại, ai hưởng lợi, ai gánh chi phí lỗi và ai có thể khiếu nại kết quả. Prediction score không quyết định hành động nào công bằng/thích hợp. Xác định intended user, người bị ảnh hưởng dù không trực tiếp dùng và bối cảnh hệ thống không nên dùng.

## 2. Fairness và tính đại diện

Data có thể thiếu đại diện một số nhóm, mã hóa quyết định quá khứ hoặc dùng proxy cho thuộc tính nhạy cảm. Accuracy tổng thể có thể che chênh lệch error rate lớn. Khi hợp pháp và phù hợp, đo data coverage cùng metric trọng yếu theo nhóm và giao cắt giữa các nhóm. Subgroup sample nhỏ tạo uncertainty; báo sample size và interval thay vì xếp hạng quá tự tin.

Fairness measure có thể xung đột. False-positive rate ngang nhau, false-negative rate ngang nhau, calibration và selection rate ngang nhau là các mục tiêu khác nhau, không phải lúc nào cũng đồng thời đạt được. Chọn tiêu chí theo bối cảnh, tham vấn stakeholder chịu ảnh hưởng và giải thích trade-off. Xóa cột nhạy cảm không xóa proxy của nó.

## 3. Privacy và data governance

Chỉ thu thập data cần cho mục đích xác định. Kiểm tra consent, permission, license, retention, access và yêu cầu deletion trước khi train. Tránh đưa secret/thông tin cá nhân vào ví dụ công khai và log. Aggregation/de-identification giảm một số rủi ro nhưng không tự loại bỏ khả năng tái định danh.

## 4. Explainability và giao tiếp

Giải thích vai trò hệ thống, input cần có, uncertainty, giới hạn đã biết và user có thể làm gì khi kết quả sai. Feature importance không phải giải thích nhân quả; lời giải thích nghe hợp lý không chứng minh model đã dùng lý do được nêu. Tránh trình bày confidence score như cam kết nếu chưa calibration/diễn giải đúng.

## 5. Human oversight và recourse

Đặt ranh giới review rõ cho ca tác động cao hoặc không chắc chắn. Reviewer cần thời gian, thông tin, thẩm quyền và đào tạo để override recommendation; một nút bấm của con người không thể thay đổi kết quả thực chất không phải oversight hiệu quả. Khi phù hợp, cung cấp đường correction/appeal và theo dõi nó có hoạt động không.

## 6. Theo dõi sau release

Theo dõi quality, hành vi subgroup, incident, feedback user và input data thay đổi. Định nghĩa alert owner/action. Đánh giá lại rủi ro khi model, data, interface hoặc bối cảnh deployment đổi. Duy trì record về version, evaluation, vấn đề đã biết và quyết định.

## Thực hành

Hình dung image classifier dùng để phân loại ảnh sản phẩm. Liệt kê người chịu ảnh hưởng, nhóm/camera/điều kiện ánh sáng thiếu trong dataset, hậu quả prediction sai, permission dữ liệu và đường human correction. Viết ba release condition phải đạt trước khi mở rộng sử dụng.

## Lỗi thường gặp

- Cho rằng một technical metric duy nhất định nghĩa fairness.
- Coi xóa protected attribute là bằng chứng model không thiên lệch.
- Thu thập dữ liệu nhạy cảm “để phòng khi cần” dù không có nhu cầu/permission.
- Gọi human review hiệu quả mà không kiểm tra reviewer có can thiệp được không.
- Công bố điểm trung bình mà không nêu giới hạn, bối cảnh và nhóm chịu ảnh hưởng.
