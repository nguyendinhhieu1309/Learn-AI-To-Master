# Context Engineering, Agent Memory và Persistence

Context của agent là thông tin sẵn có để ra quyết định tiếp theo. Memory là thông tin được giữ lại để dùng sau. Hai khái niệm có liên quan nhưng không thay thế nhau: lưu conversation không đảm bảo sẽ truy xuất đúng thông tin; truy xuất được một fact không đảm bảo fact đó mới hoặc an toàn để dùng.

## Mục tiêu học tập

- Phân tách context window của agent thành task, policy, tool, evidence và state.
- Phân biệt working, semantic, episodic và procedural memory.
- Lập budget, rút gọn, truy xuất và validate context.
- Tách workflow checkpoint khỏi long-term memory.
- Thiết kế memory có provenance, privacy, expiration và evaluation.
- Dùng human review cùng state có thể resume cho workflow hệ trọng.

## Kiến thức tiên quyết

Hoàn thành [agent workflow](01-agent-workflows-and-tools.md), [skill](03-skills-and-reusable-procedures.md), [harness design](04-harness-design-and-runtime.md) và [LangChain/LangGraph](05-langchain-and-langgraph.md).

## 1. Context được lắp cho một quyết định

Agent request có thể gồm:

- system/developer policy;
- task hiện tại của user và authorization context;
- mô tả tool khả dụng;
- conversation gần đây;
- tài liệu hoặc memory retrieved;
- kết quả trung gian và câu hỏi chưa giải quyết;
- output format cùng phần dự trữ cho câu trả lời.

Tổng input phải nằm trong context limit của model:

$$
T_{\mathrm{policy}}+
T_{\mathrm{task}}+
T_{\mathrm{tools}}+
T_{\mathrm{evidence}}+
T_{\mathrm{history}}+
T_{\mathrm{reserve}}
\leq T_{\mathrm{context}}.
$$

Giới hạn không phải mục tiêu cần lấp đầy. Context nhiều hơn có thể tăng latency/cost, gây xao nhãng và tăng nguy cơ lộ dữ liệu nhạy cảm.

## 2. Bốn loại memory

| Loại | Lưu gì | Ví dụ | Rủi ro chính |
| --- | --- | --- | --- |
| Working | task state hiện tại và tương tác gần đây | approval đang chờ, search result hiện tại | context tràn hoặc summary cũ |
| Semantic | fact tương đối ổn định | ngôn ngữ ưa thích, glossary dự án | fact sai hoặc lỗi thời |
| Episodic | sự kiện/kết quả đã xảy ra | ticket hỗ trợ trước và cách giải quyết | khái quát quá mức từ một ca |
| Procedural | cách thực hiện tác vụ | skill validate CSV | procedure lỗi thời hoặc không an toàn |

Chỉ tạo memory khi tác vụ tương lai được lợi, không phải vì có chỗ lưu. Hệ thống ghi lại mọi thứ đã đưa ra quyết định retention dù chưa ai thiết kế chính sách.

## 3. Tách workflow state khỏi long-term memory

**Workflow checkpoint:** state đủ để tiếp tục một run cụ thể tại bước an toàn. Có thể gồm task status, tool result, approval đang chờ và execution metadata.

**Long-term memory:** fact, episode hoặc procedure đã chọn có thể được retrieve ở lần chạy sau.

Hai loại có lifecycle khác nhau. Checkpoint có thể hết hạn khi workflow kết thúc; preference của user có thể tồn tại cho tới khi được sửa/xóa. Workflow checkpointer không tự động là semantic-memory system.

Khi có thể, lưu state có cấu trúc và provenance rồi render vào prompt khi cần. Đừng lưu một chuỗi prompt không cấu trúc làm record duy nhất về những gì agent biết/đã quyết.

## 4. Thiết kế memory record

Với mỗi item lưu trữ, xác định:

- content và memory type;
- source hoặc evidence span;
- owner, tenant và access-control boundary;
- thời gian tạo, xác nhận gần nhất và quy tắc expiration;
- confidence hoặc trạng thái verification;
- sensitivity class và đường xóa;
- item được model suy ra hay user nói rõ.

Ví dụ, “user thích summary ngắn” không nên âm thầm thành fact vĩnh viễn nếu chỉ suy ra từ một message ngắn. Cho phép sửa và xóa.

## 5. Chính sách đọc và ghi

### Ghi memory

Trước khi lưu:

1. Thông tin có hữu ích ngoài tác vụ hiện tại không?
2. Đây là điều user nói trực tiếp hay suy ra?
3. Có mang tính cá nhân, nhạy cảm hoặc thuộc diện quản lý không?
4. Có thể lưu dạng ngắn hơn, ít định danh hơn không?
5. User có cần duyệt hoặc xem lại được không?

Dùng ngưỡng trích xuất và provenance. Memory write nên idempotent hoặc deduplicate khi cùng fact lặp lại.

### Đọc memory

Trước khi đưa memory vào context:

1. User hiện tại có quyền xem không?
2. Nó liên quan tác vụ này không?
3. Nó còn mới và có nhất quán với evidence mới hơn không?
4. Nó có xung đột request hiện tại không?
5. Tác vụ có thể làm mà không cần nó không?

Memory truy xuất là evidence có nguồn và độ tuổi, không phải instruction ưu tiên cao hơn. Đừng để preference cũ ghi đè request hiện tại nói rõ.

## 6. Lập budget và cô đọng context

Chừa chỗ cho task hiện tại, tool schema và câu trả lời cuối trước khi thêm history. Tùy tác vụ, ưu tiên:

1. policy và authorization bắt buộc;
2. mục tiêu và constraint hiện tại của user;
3. evidence trực tiếp liên quan có provenance;
4. quyết định chưa giải quyết và workflow state;
5. phần history gần đây tối thiểu để giữ mạch;
6. background ưu tiên thấp hơn.

Khi rút gọn tương tác dài, giữ lại:

- objective và constraint;
- quyết định đã chốt;
- source reference cùng ngày tháng;
- câu hỏi/rủi ro chưa giải quyết;
- action hoàn tất và action còn pending.

Đừng tóm tắt mất phủ định, trạng thái approval, giới hạn định lượng hoặc khác biệt giữa fact quan sát và suy luận. Kiểm tra summary bằng cách resume task từ đó và xác minh action tiếp theo còn đúng.

## 7. Prompt caching như tối ưu context

Một số model provider có thể tái sử dụng prefix prompt lặp lại. Nguyên tắc chung là đặt instruction và ví dụ ổn định trước nội dung riêng theo user có thể thay đổi. Cache behavior, pricing và invalidation tùy provider; cần kiểm tra docs hiện hành trước khi dựa vào.

Không chia sẻ cached prefix giữa user nếu chứa thông tin riêng theo user/tenant. Caching cải thiện tính toán lặp, không loại bỏ yêu cầu authorization và retention control.

## 8. Persistence, interrupt và human review

Với workflow phải tạm dừng, lưu:

- run/thread identifier;
- state tối thiểu để resume;
- action chính xác đang chờ và câu hỏi approval;
- source/tool version liên quan;
- action đã chạy hay chưa.

Chỉ resume sau khi kiểm tra lại authorization và approval còn hiệu lực không. Dùng idempotency key hoặc durable action record để retry không gửi trùng payment, message hay command.

Human review đặc biệt hữu ích ngay trước side effect không đảo ngược hoặc tác động cao. Cho reviewer thấy action đề xuất, evidence hỗ trợ và hệ quả. Ghi approval, edit, reject, danh tính/vai trò reviewer và thời gian.

Time travel hoặc sửa state hữu ích khi debug, nhưng sửa production state có thể vượt business rule. Hạn chế quyền, audit và revalidate invariant sau chỉnh sửa.

## 9. Đánh giá memory như một tính năng sản phẩm

Tạo cặp ca memory có ích, không liên quan, sai, lỗi thời hoặc thuộc user khác. Đo:

- task success khi có/không có memory;
- retrieval precision và useful recall của memory;
- lỗi mâu thuẫn và stale;
- retention dữ liệu nhạy cảm và rò rỉ chéo user;
- token, latency và storage cost;
- khả năng xử lý yêu cầu sửa/xóa.

Memory làm cá nhân hóa tốt hơn nhưng gây lộ privacy không phải memory system thành công.

## Bài tập có đáp án

1. Checkpoint kiểu LangGraph có giống long-term semantic memory không?
2. User từng yêu cầu trả lời một câu. Có nên lưu thành preference vĩnh viễn?
3. Memory mâu thuẫn request hiện tại. Điều gì nên hướng dẫn phản hồi?
4. Trước khi resume workflow tạm dừng, cần kiểm tra gì?
5. Vì sao memory cần ghi thời gian và nguồn gốc?

### Đáp án

1. Không. Checkpoint phục hồi workflow state; semantic memory lưu fact tái sử dụng. Mục đích và retention khác nhau.
2. Không tự động. Coi là constraint của tác vụ đó, trừ khi lặp lại, được lưu rõ hoặc user xác nhận.
3. Thông thường request hiện tại nói rõ được ưu tiên, trong giới hạn safety/authorization policy cao hơn.
4. Authorization, độ mới của approval, trạng thái action pending, deadline và idempotency side effect.
5. Provenance và tuổi giúp phát hiện suy luận, mâu thuẫn, lỗi thời và sai phạm vi.

## Mini-project

Thiết kế memory layer cho research assistant. Định nghĩa hai memory type, schema, read/write policy, tenant boundary, expiration, đường correction/deletion và evaluation set 20 ca gồm stale/cross-user. Minh họa workflow tạm dừng rồi resume có approval check và không lặp side effect khi retry.

## Lỗi thường gặp

- Coi toàn bộ conversation transcript là memory hữu ích.
- Đặt memory retrieved cao hơn instruction hiện tại hoặc authorization.
- Lưu fact cá nhân suy luận mà không có đường review/xóa.
- Resume workflow mà không kiểm tra action đang chờ đã chạy chưa.
- Nén context mà không thử các quyết định sau khi resume.
