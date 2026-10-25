# Thiết kế Harness Agent và Runtime

## Mục tiêu học tập

Giải thích hệ thống xung quanh model call của agent, thiết kế tool loop có giới hạn và quan sát được, cải thiện harness dựa trên lỗi thực tế.

## 1. Harness là ranh giới hệ thống

Từ **harness** có thể chỉ phạm vi khác nhau. Một cách hiểu phổ biến là prompt, tool và middleware xung quanh model loop. Trong ứng dụng production, harness có thể gồm cả tạo input/context, tool dispatch, state/checkpoint, permission, sandbox, human approval, budget, logging, trace và output validation. Tài liệu thiết kế nên nêu rõ đang dùng định nghĩa nào.

Agent không chỉ là model. Hành vi quan sát được hình thành từ model, instruction, thông tin khả dụng, tool schema, runtime, policy, setting model/provider và môi trường evaluation.

~~~mermaid
flowchart LR
    User[User / API] --> Validate[Validate identity và request]
    Validate --> Context[Dựng context giới hạn và nạp skill liên quan]
    Context --> Loop[Agent loop và budget]
    Loop --> Model[Model]
    Model --> Decide{Final answer hay tool call?}
    Decide -- Tool call --> Gate[Validate argument và authorize]
    Gate --> Sandbox[Chạy tool trong môi trường giới hạn]
    Sandbox --> State[Thêm result và checkpoint state]
    State --> Loop
    Decide -- Final --> Output[Validate và trả kết quả]
    Loop --> Trace[Trace, metric và evaluation]
    Gate --> Human[Human approval với action nhạy cảm]
    Human --> Sandbox
~~~

## 2. Control loop

1. Xác thực caller và validate request.
2. Chọn instruction theo tác vụ, tool và chỉ phần context liên quan.
3. Gọi model với step/time/token/cost budget tường minh.
4. Nếu response yêu cầu tool, validate schema và authorize đúng resource/action.
5. Thực thi có timeout, isolation và idempotency; lưu structured result.
6. Thêm observation vào state rồi tiếp tục, hoặc dừng khi hoàn tất, hết budget hay gặp lỗi.
7. Validate output cuối và báo trung thực trạng thái một phần/thất bại.

Model đề xuất action. Application code tất định quyết định action có được phép và thực sự chạy những gì.

## 3. State, memory và công việc dài

Conversation history, working state và durable memory là các khái niệm khác nhau:

- **History:** message/event của hội thoại.
- **Working state:** biến tác vụ hiện tại, tiến độ, tool result và approval đang chờ.
- **Checkpoint:** giúp run phục hồi/tiếp tục sau gián đoạn.
- **Long-term memory/store:** giữ fact đã chọn qua nhiều run, cần retention, correction và access rule.
- **Artifact:** plan, task list hoặc report có thể bàn giao cho context/session mới.

Với công việc dài, chia thành đơn vị kiểm chứng được; ghi việc đã xong/tiếp theo/bị chặn; checkpoint trước hoặc sau side effect; xác minh outcome bên ngoài. Tóm tắt context giảm history nhưng không thay durable state. Session mới có handoff rõ có thể hữu ích khi việc kéo dài qua nhiều context window.

## 4. Guardrail trong harness

Thực thi giới hạn ngoài văn bản model tự do: tool allowlist, xác thực identity và resource, input/output schema, ranh giới file/network, timeout, retry cap, rate limit, spend limit và human approval cho action không thể đảo ngược. Dùng idempotency key cho operation có thể retry. Coi tài liệu retrieved và tool output là data không tin cậy.

## 5. Quan sát toàn bộ run

Trace model call, tool argument/result, state transition, approval, retry, latency và cost. Che secret, giảm thiểu dữ liệu cá nhân được lưu. Transcript đơn lẻ có thể không chứng minh action đã xảy ra; xác minh outcome trong hệ thống đích. Trace cần đủ hữu ích để phân loại lỗi ở model, context, tool, permission, state hay environment.

## 6. Cải thiện bằng experiment có kiểm soát

Bắt đầu từ kiến trúc đơn giản nhất giải quyết tác vụ. Lập evaluation set, ghi baseline success và nhóm lỗi, sau đó thêm từng thành phần harness (retrieval, validator, checkpoint hoặc specialist agent) rồi đo thay đổi. Đánh giá lại sau nâng cấp model/provider: scaffolding cũ có thể trở nên thừa hoặc tạo failure mode mới.

## Thực hành

Thiết kế research agent được tìm nguồn công khai đã duyệt và lưu bản nháp cục bộ. Nêu nguồn context, quyền tool, max step/cost, định dạng checkpoint, ranh giới approval, trường trace, validation cuối và outcome chứng minh draft tồn tại.

## Lỗi thường gặp

- Cấp tool rộng rồi dựa vào prompt để giới hạn.
- Biến mọi tác vụ thành autonomous, multi-agent, chạy dài trước khi có baseline.
- Coi summary là checkpoint bền vững của side effect và task status.
- Thêm guardrail không được đo hoặc không tích hợp với authorization của application.
- Giữ harness phức tạp sau thay đổi model mà không kiểm tra từng phần còn ích lợi không.
