# Agentic AI

Agentic system dùng model để chọn bước tiếp theo, gọi tool hoặc phối hợp workflow nhằm hoàn thành nhiệm vụ. Trong module này, agent được xem là một software system có thể đo độ tin cậy, quyền hạn và failure mode; không chỉ là prompt chạy trong vòng lặp.

## Kết quả học tập

- Phân biệt workflow cố định với agent do model điều khiển.
- Thiết kế tool interface, state và điều kiện dừng cho tác vụ giới hạn.
- Đánh giá task success, reliability, cost, latency và safety.

## Kiến thức tiên quyết

Python, API cơ bản và nhập môn NLP/language model. Học [LLM evaluation và safety](../08-nlp-llms/README.md) trước.

## Lộ trình gợi ý

1. [Agent workflow và tool](01-agent-workflows-and-tools.md)
2. [Skill và quy trình tái sử dụng](03-skills-and-reusable-procedures.md)
3. [Context engineering, memory và persistence](09-context-engineering-memory-and-persistence.md)
4. [Thiết kế harness và runtime](04-harness-design-and-runtime.md)
5. [LangChain và LangGraph](05-langchain-and-langgraph.md)
6. [MCP và interoperability](06-mcp-and-interoperability.md)
7. [Multi-agent system](07-multi-agent-systems.md)
8. [Agentic pattern và human control](10-agentic-patterns-and-human-control.md)
9. [Evaluation, security và operations](02-evaluation-security-and-operations.md)
10. [Lab agent evaluation harness](08-agent-evaluation-harness-lab.md)

## Bài thực hành

- So sánh workflow tất định với agent trên cùng tác vụ nhỏ.
- Thêm input validation, tool allowlist, step budget và điều kiện dừng rõ ràng.
- Thử tool output sai định dạng, timeout, instruction mâu thuẫn và prompt injection trong nội dung retrieval.
- Xem trace để phân biệt lỗi model, tool và dữ liệu.
- Hoàn thành [failure-injection lab](08-agent-evaluation-harness-lab.md) trước khi thêm tool có quyền ghi.

## Dự án checkpoint

Xây bounded assistant dùng một nhóm nhỏ read-only tool. Định nghĩa action được phép, test case, stopping rule và evaluation rubric. Báo task success, nhóm lỗi, cost/latency và rủi ro đã biết.
