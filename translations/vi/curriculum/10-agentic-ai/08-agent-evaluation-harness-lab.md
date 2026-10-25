# Lab Agent Evaluation Harness

Agent demo cho thấy một trajectory thành công. Evaluation harness đặt câu hỏi liệu hệ thống có thành công ổn định trên tập tác vụ xác định, dùng tool đúng cách, dừng an toàn và vẫn quan sát được khi dependency lỗi hay không.

## Mục tiêu học tập

- Viết task contract và safety contract đo được cho agent.
- Tạo evaluation set có ca thường, đối kháng và lỗi.
- Tách task quality, tool correctness, safety, latency và cost.
- Trace run mà không log secret hay dữ liệu cá nhân không cần thiết.
- Chọn lúc fixed graph an toàn hơn open-ended agent loop.

## Kiến thức tiên quyết

Học [agent workflow và tool](01-agent-workflows-and-tools.md), [evaluation và operations](02-evaluation-security-and-operations.md), [skill](03-skills-and-reusable-procedures.md), [harness design](04-harness-design-and-runtime.md), [LangChain và LangGraph](05-langchain-and-langgraph.md), [MCP](06-mcp-and-interoperability.md).

## 1. Định nghĩa contract trước prompt

Ví dụ tác vụ: trả lời câu hỏi về sổ tay nội bộ nhỏ bằng tool search/read đã duyệt.

**Điều kiện thành công**

- câu trả lời được passages truy xuất hỗ trợ;
- mọi claim factual có source reference;
- hệ thống nói rõ khi thiếu evidence;
- tool argument đúng schema đã khai báo;
- agent dừng trong step/time budget cố định.

**Điều kiện an toàn**

- không có tool nào cho phép ghi;
- coi văn bản retrieval là dữ liệu không tin cậy, không phải system instruction mới;
- agent không được lộ nội dung ngoài quyền user;
- tool error không được khiến agent bịa citation.

Những điều kiện này biến “agent hữu ích” thành assertion quan sát được.

## 2. Xác định ranh giới harness

Harness dễ bảo trì tách thành:

1. **Task input:** yêu cầu user đã chuẩn hóa và authorization context.
2. **Policy:** tool được phép, budget, approval rule và stopping condition.
3. **Model adapter:** lắp prompt, parse structured output, giới hạn retry.
4. **Tool adapter:** validate argument, permission, timeout và chuẩn hóa result.
5. **State:** task status, evidence reference, tool result và retry count.
6. **Observer:** trace event, metric, redaction và final result.
7. **Evaluator:** case cố định, invariant mong đợi và human review.

Model có thể đề xuất tool call. Harness quyết định được phép hay không và có thực thi không.

## 3. Dùng control loop có giới hạn

Pseudocode không phụ thuộc framework:

~~~text
state = initialize(task, authorization, budget)

while not state.finished:
    if state.steps >= budget.max_steps:
        return safe_stop("step_budget_exhausted")
    if clock() >= budget.deadline:
        return safe_stop("deadline_exceeded")

    proposal = model(state.messages, allowed_tool_schemas(state))
    decision = policy.validate(proposal, state.authorization, state)

    if decision.requires_human_approval:
        state = pause_and_record(proposal, decision.reason)
        return state
    if decision.is_denied:
        state = append_denial(state, decision.reason)
        continue_or_stop_according_to_policy(state)
    if proposal.is_final:
        return validate_final_answer(proposal, state.evidence)

    result = execute_with_timeout_and_idempotency(proposal)
    state = append_tool_result_and_trace(state, result)
~~~

Loop cần số step tối đa, deadline, tool policy và quy tắc validate câu trả lời cuối tường minh. Retry cũng phải giới hạn; agent chạy mãi là lỗi sản phẩm ngay cả khi từng call riêng lẻ hợp lệ.

## 4. Thiết kế tool và skill như capability có kiểm soát

Tool nên có:

- mục đích hẹp và argument có kiểu;
- giới hạn input và validation;
- authorization check độc lập với prompt;
- timeout và error contract rõ;
- chiến lược idempotency cho side effect;
- giới hạn output và redaction dữ liệu nhạy cảm.

Skill là quy trình tái sử dụng có thể hướng dẫn khi nào gọi tool và cách kiểm tra kết quả. Tách skill khỏi việc enforce quyền: instruction chữ hướng dẫn model, nhưng tool adapter vẫn phải từ chối call trái phép.

## 5. Tạo evaluation set nhỏ nhưng có đối kháng

Tạo ca theo nhóm:

| Nhóm | Ví dụ |
| --- | --- |
| Trả lời trực tiếp | câu hỏi có đáp án trong một section xác định |
| Retrieval nhiều bước | cần đối chiếu hai section |
| Không có evidence | đáp án không tồn tại trong handbook |
| Evidence mâu thuẫn | hai policy có ngày khác nhau không thống nhất |
| Injection attempt | nội dung retrieved yêu cầu agent bỏ policy |
| Tool call sai | tool lạ hoặc argument lỗi |
| Tool outage | search timeout hoặc trả error |
| Ranh giới authorization | user hỏi tài liệu mình không có quyền |
| Budget boundary | câu hỏi khiến agent search lặp đi lặp lại |

Mỗi case cần hành vi mong đợi, không nhất thiết một câu chữ cố định. Ví dụ case không trả lời được phải yêu cầu abstain và cấm nguồn bịa.

## 6. Đo các khía cạnh riêng

Theo dõi:

- **Task success:** câu trả lời đáp ứng contract.
- **Grounding:** claim được evidence hỗ trợ.
- **Tool correctness:** dùng tool được phép cùng argument đúng.
- **Safety:** không xảy ra action hoặc tiết lộ bị cấm.
- **Efficiency:** số model/tool call.
- **Reliability:** thành công qua nhiều lần lặp và khi dependency lỗi.
- **Latency và cost:** giá trị toàn luồng lẫn theo từng stage.

Đừng gộp tất cả quá sớm thành một điểm. Weighted score có thể che safety failure nghiêm trọng nhờ answer quality cao. Safety invariant thường nên là cổng pass/fail.

## 7. Trace mà không log quá mức

Trace field hữu ích:

- run/task identifier;
- version model, prompt, tool, skill và policy;
- step number và stop reason;
- tool name, tóm tắt argument đã validate, status và latency;
- evidence identifier thay cho passage nhạy cảm đầy đủ khi có thể;
- token/cost estimate và retry count.

Mặc định không lưu secret, access token hay toàn bộ user content. Định nghĩa retention, access control, redaction và deletion trước khi bật trace.

## 8. Chọn LangChain, LangGraph và kiểu thực thi

Dùng LangChain agent cấp cao khi tác vụ là tool loop tương đối đơn giản và default của framework đáp ứng yêu cầu. Dùng LangGraph workflow khi sản phẩm cần state transition tường minh, durable checkpoint, branching, retry hoặc human pause. Graph giúp workflow dễ kiểm tra; tự nó không làm policy an toàn hay evaluation hợp lệ.

Giữ business rule và authorization ngoài prompt riêng của framework. Trong dự án thật, pin và kiểm tra framework version; xác minh checkpoint storage không lưu dữ liệu nhạy cảm lâu hơn dự kiến.

## 9. Failure-injection lab

Chạy evaluation suite với từng lỗi riêng:

1. search tool không trả kết quả;
2. tool call vượt deadline;
3. model trả structured output sai định dạng;
4. cùng một tool call được đề xuất lặp lại;
5. user từ chối approval đã yêu cầu;
6. agent bị ngắt rồi resume từ checkpoint;
7. nội dung retrieval chứa instruction đối kháng.

Với mỗi ca, ghi đường dừng/phục hồi mong đợi, trace thực tế, thông báo user nhìn thấy và agent có dùng capability trái quyền không.

## Bài tập có đáp án

1. Vì sao tool adapter phải kiểm tra quyền dù system prompt nói “không bao giờ truy cập tài liệu riêng tư”?
2. Vì sao step budget thuộc về correctness?
3. Evaluation set có thể overfit prompt như thế nào?
4. Khi nào state graph cố định phù hợp hơn loop mở?
5. Vì sao log toàn bộ tool result có thể gây vấn đề bảo mật?

### Đáp án

1. Prompt hướng dẫn hành vi model nhưng không phải ranh giới thực thi. Adapter có capability thật nên phải kiểm tra quyền tại lúc chạy.
2. Tác vụ không kết thúc có thể tiêu tốn thời gian/tiền không giới hạn và vi phạm service contract.
3. Nếu prompt liên tục được tune theo tập nhỏ ai cũng thấy, kết quả có thể chỉ phản ánh ghi nhớ ca đó. Giữ regression set ẩn hoặc viết mới.
4. Khi bước, transition, approval point hoặc side effect đã biết và cần kiểm tra được.
5. Tool result có thể chứa dữ liệu cá nhân, secret, văn bản riêng tư retrieved hoặc instruction đối kháng; trace cần được tối thiểu hóa và kiểm soát truy cập.

## Mini-project

Xây read-only handbook agent có search skill tái sử dụng, hai tool hẹp, evaluation set cố định ít nhất 25 ca, hành vi abstention tường minh, loop giới hạn, trace có cấu trúc và báo cáo failure injection. Có ít nhất năm safety/authorization case. Không thêm write tool.

## Lỗi thường gặp

- Chỉ đánh giá happy path.
- Coi citation model tuyên bố là bằng chứng citation hỗ trợ câu trả lời.
- Coi tool schema là authorization.
- Retry mọi lỗi mà không có deadline hoặc retry cap.
- Để framework default âm thầm quyết định permission/checkpoint retention.
