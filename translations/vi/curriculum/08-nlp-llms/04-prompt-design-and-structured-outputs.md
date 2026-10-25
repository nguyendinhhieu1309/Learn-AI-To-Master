# Thiết kế Prompt và Structured Output

## Mục tiêu học tập

Viết prompt có ranh giới nhiệm vụ rõ ràng, tách trusted instruction khỏi dữ liệu không tin cậy và validate đầu ra model dành cho máy đọc.

## 1. Prompt là một phần của chương trình

Prompt hữu ích thường nêu: nhiệm vụ, ý nghĩa đầu vào, ràng buộc, định dạng đầu ra và cách xử lý khi thiếu evidence. Đưa một số ví dụ đại diện nếu chúng làm rõ edge case. Giữ system/developer instruction tách khỏi văn bản do user cung cấp và nội dung retrieval.

Tránh yêu cầu mơ hồ kiểu “hãy hoàn hảo”. Thay bằng tiêu chí quan sát được: đưa source ID, trả về schema cố định, abstain khi thiếu trường bắt buộc hoặc hỏi một câu làm rõ.

## 2. Đánh dấu dữ liệu, nhưng vẫn xem là không tin cậy

Đánh dấu rõ điểm bắt đầu và kết thúc của tài liệu hoặc user payload; yêu cầu model xử lý nó như dữ liệu. Delimiter giúp ranh giới sáng tỏ nhưng không phải cơ chế phân quyền. Tài liệu độc hại vẫn có thể chứa chỉ thị nhằm điều hướng model; application code phải thực thi permission và tool rule.

## 3. Đầu ra có cấu trúc

Với dữ liệu đi vào code downstream, yêu cầu output có schema (ví dụ JSON hoặc typed model) nếu provider hỗ trợ. Sau đó parse và validate. Schema hợp lệ chỉ đảm bảo dạng và kiểu dữ liệu, không đảm bảo sự thật hoặc hành động được cho phép.

~~~python
from pydantic import BaseModel, Field

class TicketSummary(BaseModel):
    category: str = Field(description="One of billing, technical, or other")
    needs_human_review: bool
    summary: str
~~~

Sau khi nhận output, kiểm tra category theo allowlist và áp dụng authorization logic trước khi hành động. Trả về lỗi validation có thể xử lý hoặc yêu cầu sinh lại với số lần thử giới hạn; không âm thầm tự bịa trường còn thiếu.

## 4. Prompt injection và ranh giới tin cậy

Prompt injection là nỗ lực tác động hành vi model qua văn bản không tin cậy. Coi user input, trang truy xuất, email, code và tool output là dữ liệu. Không đặt secret trong prompt nếu không thật sự cần; không cấp tool quyền rộng chỉ vì instruction nói rằng cần. Tool executor phải tự kiểm tra danh tính, permission và tham số.

## 5. Version và kiểm thử prompt

Lưu prompt cùng version và đánh giá trên tập cố định gồm trường hợp thường, mơ hồ, không thể trả lời và đối kháng. So sánh đặc tính đầu ra chứ không chỉ ví dụ được chọn sẵn. Sửa prompt có thể ảnh hưởng lựa chọn tool, refusal, độ dài và latency; regression-test từng hành vi ứng dụng cần.

## Thực hành

Viết prompt trích tổng tiền hóa đơn. Nêu currency, cách tính thuế, các trường output và xử lý nếu hai tổng tiền mâu thuẫn. Tạo test cho currency bị thiếu, nhiều trang, OCR sai và instruction bị chèn trong hóa đơn. Xác định bước nào phải được kiểm tra bằng code chứ không chỉ bằng prompt.

## Lỗi thường gặp

- Nghĩ JSON output đồng nghĩa nội dung chính xác.
- Đặt tài liệu user sau trusted instruction mà không đánh dấu là dữ liệu không tin cậy.
- Dùng ngôn từ prompt để thực thi access control.
- Đổi prompt trên production mà không version hóa và regression evaluation.
