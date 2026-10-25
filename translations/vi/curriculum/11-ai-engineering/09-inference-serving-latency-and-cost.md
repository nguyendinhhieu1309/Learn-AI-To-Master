# Inference Serving, Latency và Cost

Model quality chỉ là một phần AI system. Service hữu ích phải đáp ứng yêu cầu latency, throughput, memory, availability và cost dưới traffic thực tế.

## 1. Định nghĩa service objective

Nêu request size, traffic shape, concurrency, hardware, quality constraint và latency objective. Báo percentile latency như p50, p95, p99; average che tail behavior. Tách thời gian queue, preprocessing, model execution, tool/retrieval call, serialization và network.

Với autoregressive generation, time-to-first-token và token generation rate là hai metric riêng. Với batch inference, throughput và thời gian hoàn thành từ đầu đến cuối quan trọng. Chọn metric theo trải nghiệm user.

## 2. Amdahl's law và bottleneck

Nếu tỷ lệ công việc $p$ được tăng tốc lên hệ số $s$, total speedup:

$$
S = \frac{1}{(1-p)+p/s}
$$

Tăng tốc neural network ít tác dụng nếu retrieval, database access, CPU preprocessing hoặc queueing là bottleneck. Profile từng stage trước khi tối ưu.

## 3. Batching và concurrency

Batching chia sẻ chi phí compute cố định và cải thiện accelerator utilization nhưng tăng queue delay, có thể lãng phí với request độ dài khác nhau. Dynamic batching cần maximum wait time và maximum batch size. Với language model, continuous batching tận dụng thiết bị khi sequence bắt đầu/kết thúc ở thời điểm khác nhau.

Giới hạn concurrency và queue. Queue không giới hạn biến overload thành latency cực cao và tiêu thụ memory. Dùng backpressure, deadline, admission control và thông báo overload rõ.

## 4. Tối ưu model và request

- **Quantization** giảm độ chính xác weight và memory; validate task quality cùng hỗ trợ hardware.
- **Distillation** chuyển hành vi sang model nhỏ; đánh giá student trên slice quan trọng.
- **Pruning** giảm parameter nhưng tốc độ chỉ tăng khi sparse kernel hỗ trợ.
- **Caching** giúp request lặp nhưng cache key, freshness, privacy boundary và invalidation phải đúng.
- **Speculative decoding** tăng tốc sinh token nếu draft model nhỏ được chấp nhận đủ thường xuyên.

Đo quality sau mỗi tối ưu; compression ratio tốt nhất vô ích nếu làm hỏng ca hiếm nhưng quan trọng.

## 5. Ước lượng capacity và cost

Ước lượng request/giây cao điểm, token/example mỗi request, concurrency, memory mỗi model replica, accelerator utilization và redundancy khi lỗi. Tính model load time, warm-up và autoscaling delay. Với language model, cả input/output token ảnh hưởng cost và latency.

So cost trên mỗi task thành công, không chỉ cost/request. Model rẻ hơn nhưng cần retry nhiều hoặc human correction có thể đắt hơn tổng thể.

## 6. Reliability và suy giảm an toàn

Định nghĩa timeout/retry policy theo operation. Retry tool action không idempotent có thể tạo side effect trùng. Dùng circuit breaker hoặc fallback khi dependency lỗi; fallback phải giữ hành vi an toàn như abstain hoặc escalation cho người. Không để fallback âm thầm bỏ qua access control/validation.

Thử cold start, dependency outage, model-load failure, response sai định dạng, cạn tài nguyên và streaming lỗi một phần. Giữ rollback path và artifact đã biết tốt.

## 7. Benchmark đúng cách

Warm up runtime, pin model/hardware version, dùng input/output length đại diện, thử concurrency thực tế và báo tail latency cùng throughput. Với generative model, tách prefill khỏi decode. Không so số đo từ batch, quantization, context length hoặc hardware khác nhau mà không nói rõ.

## Bài tập

1. Network inference chiếm 20% request time và nhanh gấp bốn lần. Tốc độ toàn hệ thống tăng tối đa bao nhiêu theo Amdahl's law?
2. Batching tăng throughput nhưng vi phạm p99 latency. Control nào có thể cân bằng mục tiêu?
3. Quantization giảm serving cost 40% nhưng làm hại slice hiếm, quan trọng với safety. Promotion cần điều kiện gì?
4. Tool request timeout và service tự retry. Cần thuộc tính nào để tránh side effect trùng?
