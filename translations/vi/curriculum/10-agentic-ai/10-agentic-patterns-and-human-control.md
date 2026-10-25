# Agentic Pattern và Human Control

Agent pattern là lựa chọn control flow. ReAct, planning, reflection, router và multi-agent handoff đánh đổi tính linh hoạt với latency, cost, reliability. Chọn pattern nhỏ nhất đáp ứng task contract.

## Mục tiêu học tập

- Mô tả ReAct, plan-and-execute, reflection, routing và delegation.
- Quyết định lúc nào parallel tool hoặc specialist agent giúp ích.
- Biểu diễn approval và pause/resume thành state transition tường minh.
- Ngăn approval lặp/lỗi thời gây side effect trùng.
- Đánh giá cả trajectory, không chỉ văn bản cuối.

## Kiến thức tiên quyết

Hoàn thành [agent workflow](01-agent-workflows-and-tools.md), [LangChain/LangGraph](05-langchain-and-langgraph.md), [multi-agent system](07-multi-agent-systems.md), [context, memory và persistence](09-context-engineering-memory-and-persistence.md).

## 1. ReAct: xen kẽ dùng tool và quan sát

ReAct xen kẽ chọn action, quan sát kết quả và chọn action kế tiếp. Trace thực tế có thể ghi:

1. subgoal hiện tại;
2. tool và argument đã validate được đề xuất;
3. tool result hoặc lỗi;
4. evidence mới thu được;
5. action tiếp theo hoặc lý do dừng.

Trace có cấu trúc này đủ để review hành vi. Không yêu cầu hoặc làm lộ private chain-of-thought; thay vào đó ghi lý do ngắn có thể audit, evidence và action.

Dùng ReAct khi lựa chọn tool tiếp theo phụ thuộc kết quả tool trước. Thêm step budget, deadline, allowed-tool set và điều kiện kết thúc rõ ràng.

## 2. Plan and execute

Planner chia task thành bước; executor thực hiện. Cách này tăng độ bao phủ tác vụ dài và giúp nhìn thấy tiến độ, nhưng plan có thể sai/lỗi thời khi observation thay đổi.

Planner vững cần:

- sinh step có kiểu, giới hạn số lượng;
- chỉ dependency và evidence mong đợi;
- đánh dấu step cần approval;
- cho phép lập kế hoạch lại khi evidence mâu thuẫn giả định;
- dừng khi xong hoặc không thể tiếp tục an toàn.

Không để plan do model sinh trực tiếp cấp quyền side effect. Mọi action vẫn phải qua policy và tool validation.

## 3. Reflection và critique

Reflection step kiểm tra draft/trajectory theo tiêu chí tường minh: completeness, evidence support, schema validity, safety hoặc task constraint. Với rule kiểm tra được tất định, validator riêng có thể đáng tin hơn.

Reflection chỉ giúp khi:

- critique rubric cụ thể;
- hệ thống nhận diện được vấn đề cần sửa;
- retry được giới hạn;
- evaluator không chỉ lặp lại sự tự tin thiếu bằng chứng của cùng model.

Đặt hard stop sau số lần sửa cố định; báo uncertainty còn lại thay vì lặp vô hạn.

## 4. Routing và specialist

Router chọn workflow/tool chuyên biệt theo request. Chuyên môn hóa tăng tập trung nhưng cần routing đáng tin và trách nhiệm tổng hợp rõ.

Parallel expert call hữu ích khi:

- subtask độc lập;
- mỗi expert có role và evidence scope hẹp;
- bước tổng hợp cuối giữ được bất đồng và provenance.

Delegation tuần tự phù hợp khi bước sau phụ thuộc kết quả trước. Parallelism có thể giảm wall time nhưng tăng token cost, retrieval trùng và câu trả lời mâu thuẫn.

Đừng tạo agent riêng cho từng tool. Deterministic function rẻ và dễ kiểm tra hơn khi bước đã biết.

## 5. Human-in-the-loop state machine

Biểu diễn action hệ trọng thành state rõ:

| State | Ý nghĩa | Bước tiếp theo được phép |
| --- | --- | --- |
| drafted | action đề xuất đã chuẩn bị | validate, sửa hoặc từ chối |
| pending_approval | đang chờ reviewer có quyền | approve, sửa, từ chối hoặc hết hạn |
| approved | reviewer đã duyệt version action cụ thể | kiểm tra lại rồi thực thi một lần |
| executed | side effect hoàn tất và đã ghi nhận | báo kết quả |
| rejected / expired | action không được chạy | dừng hoặc tạo đề xuất mới |

Approval gắn với payload action cụ thể. Nếu payload đổi sau khi duyệt, phải xin approval mới. Lưu action ID và idempotency key.

## 6. Checkpoint, resume, replay và edit

Checkpoint chỉ nên lưu state tối thiểu cần để tiếp tục an toàn. Khi resume:

1. nạp state từ đúng user/thread;
2. xác minh authorization và policy hiện hành;
3. kiểm tra approval còn hiệu lực không;
4. xác định external action đã xảy ra chưa;
5. thực thi tối đa một lần bằng idempotency;
6. ghi state mới và trace.

Replay hữu ích để debug nhưng có thể gọi tool lặp lại nếu side effect không cô lập. Dùng dry-run tool hoặc recorded tool output khi replay. Sửa state có thể gỡ process kẹt nhưng cần audit record và chạy lại invariant sau chỉnh sửa.

## 7. Chọn pattern

| Đặc tính tác vụ | Pattern khởi đầu |
| --- | --- |
| Trình tự cố định, không cần chọn tool bất định | deterministic workflow |
| Lựa chọn tool phụ thuộc observation | bounded ReAct loop |
| Nhiều subtask phụ thuộc nhau | plan-and-execute có replanning |
| Draft cần kiểm tra độc lập | generation rồi validator/critic |
| Phân tích chuyên gia độc lập | parallel call chuyên biệt rồi tổng hợp |
| Action không đảo ngược/tác động cao | state human approval tường minh |

Có thể kết hợp pattern, nhưng mỗi loop/delegation boundary thêm một bề mặt lỗi và evaluation mới.

## 8. Đánh giá trajectory

Với từng run, kiểm tra:

- tác vụ đã hoàn thành chưa;
- mỗi tool call có cần thiết và được phép không;
- observation có khiến action sau thay đổi hợp lý không;
- evidence có hỗ trợ phản hồi cuối không;
- stopping condition có hoạt động không;
- retry, delegation và review có nằm trong budget không;
- final action có khớp payload đã duyệt không.

Thành công ở câu trả lời cuối có thể che đường đi không an toàn. Cần kiểm tra tool và approval ở cấp trajectory.

## Bài tập có đáp án

1. Plan yêu cầu gửi email khách hàng sau khi tra cứu. Cần điều kiện gì trước khi gửi?
2. Khi nào parallel expert analysis là lựa chọn kém?
3. Vì sao approval phải gắn với payload cụ thể?
4. Replay có thể làm action bên ngoài lặp lại thế nào?
5. Reflection hết retry limit thì nên làm gì?

### Đáp án

1. Kiểm tra permission/policy, approval còn hợp lệ nếu cần, xác thực lại recipient/content và thực thi idempotent.
2. Khi subtask phụ thuộc kết quả nhau, cost hạn chế hoặc không thể giải quyết bất đồng output đáng tin.
3. Nếu không, action khác hoặc đã sửa có thể chạy dưới approval dành cho đề xuất cũ.
4. Tool node replay có thể gửi lại message hoặc giao dịch nếu side effect không được giả lập/idempotent.
5. Dừng, báo vấn đề chưa giải quyết hoặc chuyển người review; không tiếp tục loop vô hạn.

## Mini-project

Xây research workflow nhỏ: retrieve evidence, nhờ hai specialist phân tích câu hỏi độc lập, tổng hợp điểm bất đồng và pause trước side effect bên ngoài. Thêm state diagram, max-step policy, kế hoạch checkpoint/resume test, idempotency strategy và rubric đánh giá trajectory.

## Lỗi thường gặp

- Coi plan hợp lý là authorization để thực thi.
- Nhờ model critique chính nó mà không có rubric độc lập.
- Chạy parallel agent có phạm vi chồng lấn và mâu thuẫn.
- Resume approval cũ sau khi payload action đã đổi.
- Replay production trace với tool đang gây side effect thật.
