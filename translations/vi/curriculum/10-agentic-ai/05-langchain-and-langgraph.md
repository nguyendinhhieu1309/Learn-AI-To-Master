# LangChain và LangGraph

## Mục tiêu học tập

Chọn giữa agent framework cấp cao và explicit graph runtime; xây tool-calling agent nhỏ; hiểu checkpoint và human approval.

> **Lưu ý phiên bản:** Framework API thay đổi nhanh. Trước khi chép ví dụ vào dự án, hãy kiểm tra version đã cài và tài liệu tương ứng.

## 1. Các thành phần kết hợp ra sao?

LangChain cung cấp tích hợp model/tool và agent pattern cấp cao. API hiện hành create_agent đóng gói model-and-tool loop và có thể mở rộng bằng middleware. LangChain mô tả agent loop này chạy trên LangGraph ở bên dưới.

LangGraph là framework/runtime orchestration cấp thấp hơn cho workflow tường minh có state. Graph có **state**, các **node** đọc/cập nhật state và **edge** điều khiển bước kế tiếp. Nó hỗ trợ kết hợp node tất định với node do model điều khiển, loop, persistence, streaming và human-in-the-loop pause. Có thể dùng LangGraph mà không cần LangChain.

LangSmith là sản phẩm tùy chọn để trace/evaluation và các công việc vòng đời liên quan trong hệ sinh thái này. Có thể kết nối hệ thống tracing/evaluation khác.

## 2. Tool agent nhỏ bằng LangChain

Ví dụ theo hình dạng API Python hiện tại; với dự án thật, cài provider integration và pin version tương thích. Tên model chỉ là placeholder vì định danh của provider thay đổi.

~~~python
from langchain.agents import create_agent
from langchain.tools import tool

@tool
def lookup_order(order_id: str) -> str:
    """Return a read-only status for one order ID."""
    # Replace with an authorized database lookup; never trust the ID alone.
    return f"Order {order_id}: status unavailable in this example"

agent = create_agent(
    model="provider:model",
    tools=[lookup_order],
    system_prompt="Use the read-only lookup when the user asks about an order."
)

result = agent.invoke({
    "messages": [{"role": "user", "content": "Check order A-1041."}]
})
~~~

Type hint và docstring truyền schema/mục đích tool cho framework/model. Application vẫn phải validate identity, tenant ownership, timeout và dữ liệu trả về. Tool call đúng kiểu vẫn có thể không được cấp quyền hoặc sai về ý nghĩa.

Middleware có thể kiểm tra/thay đổi quá trình thực thi quanh model/tool step. Ứng dụng thường gặp: logging, thay prompt/context, retry/fallback, rate limit, PII check, summarization và human approval. Kiểm tra đúng thứ tự chạy và middleware áp dụng lên nhánh nào trong version framework của bạn.

## 3. Khi nào dùng trực tiếp LangGraph?

Dùng explicit graph khi workflow có stage tất định quan trọng, nhiều route, loop cần giới hạn, parallel branch, điểm pause/resume để duyệt hoặc durable state. Giữ tác vụ request-response đơn giản ở dạng đơn giản; graph hữu ích khi làm hành vi rõ và dễ kiểm soát hơn.

~~~python
from typing_extensions import TypedDict
from langgraph.graph import StateGraph, START, END

class State(TypedDict):
    question: str
    evidence: str
    answer: str

def retrieve(state: State):
    return {"evidence": "Evidence retrieved for: " + state["question"]}

def answer(state: State):
    return {"answer": f"Answer from evidence: {state['evidence']}"}

builder = StateGraph(State)
builder.add_node("retrieve", retrieve)
builder.add_node("answer", answer)
builder.add_edge(START, "retrieve")
builder.add_edge("retrieve", "answer")
builder.add_edge("answer", END)
graph = builder.compile()
~~~

Graph này dùng node retrieve/answer giả lập để nhìn rõ control flow. Hãy thay bằng thành phần thật và định nghĩa validation/error path; không được coi placeholder text là evidence.

## 4. Persistence và human-in-the-loop

LangGraph checkpointer lưu graph state theo thread; store có thể giữ application data xuyên thread. In-memory saver hữu ích trong bài học nhưng không tồn tại sau khi process khởi động lại. Production cần persistent backend phù hợp, retention policy, access control và quản lý kích thước state.

Interrupt có thể tạm dừng thực thi và chờ input bên ngoài. Resume dùng cùng thread ID và resume value. Node có thể chạy lại từ đầu sau resume, vì vậy code trước interrupt phải an toàn khi chạy lặp; đặt side effect sau approval/execution boundary và dùng idempotency.

## 5. Chọn framework theo nhu cầu

| Nhu cầu | Có thể cân nhắc |
| --- | --- |
| Model + tool loop phổ biến và tích hợp sẵn | LangChain agent abstraction |
| State machine tường minh, branching, persistence, pause/resume | LangGraph |
| Primitive Python gọn cho agent, tool, handoff, guardrail và trace | OpenAI Agents SDK |
| Chuẩn kết nối giữa AI host với tool/resource bên ngoài | MCP |
| Instruction có thể tái sử dụng theo tác vụ và asset tùy chọn | Agent Skills |

Các nhóm này có phần giao nhau nhưng không thay thế lẫn nhau. Bắt đầu ở layer nhỏ nhất đáp ứng mức kiểm soát cần có. Khi có thể, giữ task logic và evaluation độc lập framework.

## Thực hành

Triển khai workflow “search → draft answer → validate citation → human approval → publish”. Đánh dấu bước nào phải tất định, bước nào dùng LLM, bước nào cần persistent state và bước nào phải pause. Vẽ graph trước khi code.

## Lỗi thường gặp

- Coi LangChain và LangGraph là hai tên đồng nghĩa cạnh tranh: một bên cung cấp component cấp cao, bên kia điều phối/runtime tường minh.
- Dùng in-memory checkpointer làm production memory lâu dài.
- Coi middleware hoặc tính năng human approval của framework là thiết kế bảo mật hoàn chỉnh.
- Chép ví dụ code mà không pin thư viện/provider version hoặc kiểm tra docs hiện hành.
