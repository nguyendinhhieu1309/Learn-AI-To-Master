# Model Serving và Inference Pattern

## Mục tiêu học tập

Chọn inference pattern, định nghĩa model-serving contract và đo toàn bộ request path dưới tải thực tế.

## 1. Batch, synchronous và asynchronous

- **Batch inference:** xử lý dataset hoặc queue đã biết theo lô. Throughput hiệu quả quan trọng; kết quả có thể trả sau.
- **Synchronous online inference:** caller chờ phản hồi. Latency, availability và giới hạn request quan trọng.
- **Asynchronous job:** API trả job ID; công việc tiếp tục trong queue và caller poll/subscribe để biết hoàn tất. Hữu ích với job dài, file lớn hoặc human review.
- **Streaming response:** trả token/event từng phần khi được tạo. Cải thiện thời gian cảm nhận tới kết quả đầu tiên, nhưng cần cancellation, buffering và ngữ nghĩa lỗi một phần.

Chọn theo trải nghiệm user và ràng buộc vận hành. Đừng cung cấp synchronous endpoint cho tác vụ thường xuyên vượt caller timeout.

## 2. Đường đi của serving

Inference request có thể đi qua authentication → validation → preprocessing/tokenization → queue → model runtime → postprocessing → output validation → response/logging. Đo từng stage. Database lookup hoặc giải mã image chậm có thể chiếm phần lớn model latency.

Định nghĩa input/output schema, model/preprocessing version, unit, error case, timeout, payload tối đa và ý nghĩa response. Thêm request/correlation ID để nối log/trace mà không lộ nội dung nhạy cảm thô.

## 3. Throughput và latency

Latency là thời gian mỗi request; throughput là số request/example trên đơn vị thời gian. Batching xử lý nhiều input cùng lúc có thể cải thiện accelerator utilization nhưng làm request đầu chờ batch đầy. Đo p50, p95, p99 latency dưới concurrency dự kiến, không chỉ một call trên máy cá nhân.

Capacity planning xét model memory, accelerator memory, token/image size, batch size, concurrency, queue length và warm-up. Với LLM, input length và số token sinh ảnh hưởng mạnh tới thời gian/chi phí. Với vision, độ phân giải image ảnh hưởng compute/memory.

## 4. Scale và optimize theo thứ tự

Bắt đầu bằng baseline service đúng. Profile trước khi optimize. Có thể cache kết quả tất định an toàn, batching, model nhỏ hơn, quantization, runtime tối ưu, request routing và horizontal scaling. Cache key cần gồm version model, prompt, preprocessing, tenant/permission và data có liên quan; không bao giờ trả kết quả riêng của user này cho user khác.

## 5. Backpressure và lỗi có kiểm soát

Dùng queue có giới hạn, request limit, timeout, cancellation và load shedding để tránh overload lan rộng. Định nghĩa fallback: trả kết quả một phần, xếp job vào queue, yêu cầu user thử lại, dùng model đơn giản hơn hoặc chuyển sang người. Fallback cần rõ ràng và quan sát được.

## Thực hành

Thiết kế image-classification API cho đợt tăng lên 100 request/giây. Nêu synchronous hay asynchronous, image size tối đa, queue policy, mục tiêu p95 latency, tín hiệu scale, timeout và xử lý khi hết GPU capacity.

## Lỗi thường gặp

- Chỉ đo thời gian model chạy thay vì end-to-end latency.
- Tăng replica mà không kiểm tra memory, queue hoặc quota provider.
- Retry vô hạn khi overload.
- Cache prediction mà không đưa model/data/permission version vào cache key.
