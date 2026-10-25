# Agent Workflow và Tool

## Mục tiêu học tập

Chọn giữa workflow cố định và agent, thiết kế tool interface hẹp, giới hạn vòng lặp do model điều khiển.

## 1. Workflow hay agent?

**Workflow** chạy theo trình tự định trước: lấy dữ liệu, tóm tắt, validate schema, trả kết quả. **Agent** dùng model chọn một hay nhiều action tiếp theo dựa trên quan sát. Workflow thường dễ kiểm thử và vận hành hơn. Chỉ dùng agent khi quyết định linh hoạt tạo giá trị mà cách đơn giản không xử lý được.

## 2. Xác định ranh giới tác vụ

Nêu rõ nhiệm vụ, tool được phép, thông tin hệ thống có thể truy cập, output mong muốn và điều kiện dừng. Mỗi lần chạy cần step budget, time limit và token/cost limit. Quy định cách xử lý khi model từ chối, trả output không hợp lệ hoặc lặp action.

## 3. Thiết kế tool

Tool nên có mục đích hẹp và schema có kiểu. Validate argument trước khi thực thi và validate kết quả trước khi gửi lại model. Thêm timeout và error rõ ràng. Ưu tiên action idempotent (gọi lặp lại không gây hiệu ứng thêm) khi retry. Áp dụng least privilege: search chỉ đọc an toàn hơn quyền shell hay filesystem tùy ý.

Ví dụ hợp đồng tool:

~~~text
search_docs(query: string, top_k: integer 1..5)
  -> list of {title, excerpt, source_id}
~~~

Giữ credential ngoài prompt. Structured tool call vẫn là output model không tin cậy; permission phải được thực thi trong program code.

## 4. Vòng lặp có giới hạn

~~~mermaid
flowchart TD
    Request[Yêu cầu đã validate] --> Decide[Model chọn bước tiếp theo]
    Decide --> Stop{Trả lời cuối hay dừng?}
    Stop -- Có --> Validate[Validate kết quả]
    Validate --> Done[Trả kết quả]
    Stop -- Không --> Gate{Tool được phép và còn budget?}
    Gate -- Không --> Safe[Dừng với kết quả một phần an toàn]
    Gate -- Có --> Tool[Chạy tool có timeout]
    Tool --> Observe[Validate và ghi nhận kết quả]
    Observe --> Decide
~~~

~~~text
state = yêu cầu user đã validate
for step trong 1..MAX_STEPS:
    decision = model(state, allowed_tools)
    nếu decision là final: validate rồi trả về
    nếu tool không được phép: dừng an toàn
    result = chạy tool với timeout và permission check
    state = thêm observation và tool result
trả kết quả chưa hoàn chỉnh một cách rõ ràng
~~~

Ghi đủ để debug (model/version, tool name, timing, outcome) nhưng giảm thiểu dữ liệu nhạy cảm. Không bao giờ coi tài liệu retrieved là instruction đáng tin.

## 5. State và memory

Context không đồng nghĩa memory đáng tin cậy. Chỉ lưu thông tin cần cho tác vụ, với quy tắc ownership, retention và deletion rõ ràng. Summary có thể bỏ hoặc đổi dữ kiện; giữ source reference cho claim quan trọng. Khi có thể, biểu diễn state machine tường minh.

## Thực hành

Thiết kế agent tìm tài liệu công khai và soạn trả lời nhưng không thể ghi/gửi message. Nêu tool, schema, max step, xử lý lỗi và validate output cuối. So sánh với workflow retrieval-then-answer cố định.

## Lỗi thường gặp

- Cấp tool rộng khi vài function chuyên biệt đã đủ.
- Để vòng lặp, retry hoặc chi phí không giới hạn.
- Coi JSON hợp lệ là bằng chứng action an toàn.
- Che giấu lỗi hoặc tuyên bố thành công bịa đặt sau tool timeout.
