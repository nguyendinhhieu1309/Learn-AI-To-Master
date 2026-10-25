# Video, tracking và deployment

[Xem bài gốc bằng tiếng Anh](../../../../curriculum/06-computer-vision/03-video-tracking-and-deployment.md)

## Mục tiêu học tập

Xử lý video như stream có thứ tự thời gian, phân biệt detection với tracking và đo hệ thống vision dưới ràng buộc compute/camera thực tế.

## 1. Video là một chuỗi

Video gồm frame có timestamp và frame rate. Xử lý mọi frame tốn hơn lấy mẫu. Nếu object di chuyển nhanh, lấy mẫu thưa có thể bỏ lỡ nó hoặc khiến chuyển động nhìn thấy quá lớn. Motion blur, rolling shutter và frame rơi ảnh hưởng kết quả. Giữ timestamp thay vì giả định khoảng cách giữa frame luôn chính xác.

## 2. Detection và tracking

Detector trả lời “frame này có object nào?”. Tracker cố giữ identity qua thời gian. Pipeline tracking-by-detection đơn giản chạy detector mỗi frame rồi ghép box bằng overlap, appearance, dự đoán motion hoặc kết hợp. Occlusion, cảnh đông, object giống nhau, camera motion và detection bị hụt có thể gây identity switch.

Đánh giá cả detection và identity theo thời gian. Xem track đứt đoạn và ID switch, không chỉ box score từng frame. Metric như IDF1 hoặc HOTA tóm tắt khía cạnh khác nhau; nêu metric và version dùng.

## 3. Thông tin thời gian

Optical flow ước lượng chuyển động pixel thấy được giữa frame. Nó hỗ trợ phát hiện motion/stabilize video nhưng nhạy với texture, thay đổi sáng và displacement lớn. Temporal model có thể dùng nhiều frame nhưng cần video đại diện và split cẩn thận; frame cùng bản ghi thường nên ở cùng split.

## 4. Ngân sách real-time

Đo latency từ lúc capture tới kết quả, không chỉ inference time của neural network. Tính cả decode ảnh, resize, preprocessing, chuyển sang accelerator, inference, postprocessing và truyền thông. Báo median, tail latency và tỷ lệ rơi frame. Model trung bình 20ms nhưng đôi lúc mất 400ms có thể không phù hợp vòng lặp real-time.

Với hardware hạn chế, cân nhắc giảm resolution input, quantization, đánh đổi batching hoặc architecture nhẹ hơn. Đánh giá lại quality sau mỗi thay đổi nén/runtime.

## 5. Checklist deployment

- Camera và runtime dùng cùng channel order, normalization, crop và resize.
- Camera calibration khớp lens, focus và resolution đang dùng cho tác vụ hình học.
- Threshold được chọn bằng validation data và lưu cùng model.
- Kiểm tra performance trong ánh sáng, blur, occlusion và camera motion dự kiến.
- Có fallback rõ khi prediction cũ, thiếu hoặc confidence thấp.

## Thực hành

Chọn video ngắn. So detection từng frame với detection mỗi frame thứ năm. Ghi runtime, event bị bỏ lỡ và thay đổi association. Giải thích vì sao lựa chọn nhanh hơn có thể hoặc không thể chấp nhận cho ứng dụng.

## Lỗi thường gặp

- Chia ngẫu nhiên frame kề nhau vào train và test.
- Xem prediction mỗi frame như track độc lập.
- Chỉ báo inference time trung bình.
- Nén model mà không đo đổi chất lượng trên ví dụ khó.
