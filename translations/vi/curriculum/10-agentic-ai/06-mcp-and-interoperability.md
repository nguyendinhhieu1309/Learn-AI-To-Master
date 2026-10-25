# MCP và Agent Interoperability

## Mục tiêu học tập

Giải thích Model Context Protocol (MCP) chuẩn hóa điều gì, phân biệt ba primitive chính và kết nối server mà không nhầm interoperability với authorization.

## 1. Bài toán tích hợp

Không có protocol chung, mỗi AI application có thể phải viết adapter riêng cho từng database, filesystem, ticket system hoặc service. MCP định nghĩa client-server protocol để cung cấp capability và context cho AI application. Host thường quản lý model và trải nghiệm user; MCP client kết nối tới MCP server để lộ capability.

MCP không làm model thông minh hơn, không định nghĩa toàn bộ agent workflow và không chứng minh tool an toàn. Nó chuẩn hóa cách khám phá và gọi capability; application vẫn phải quyết định tin server nào, lộ capability nào và user nào được dùng.

## 2. Ba primitive thường gặp

- **Tools:** function có thể thực thi mà model gọi, như tìm ticket hoặc tạo bản nháp.
- **Resources:** dữ liệu application/client chọn cung cấp, như tài liệu hoặc record có cấu trúc.
- **Prompts:** message template tái sử dụng được đưa cho user/application gọi.

Phân biệt này quan trọng với control. Tool call do model yêu cầu có thể tạo side effect; resource là context application cung cấp; prompt là interaction template. Cần review tên, schema, error và content trước khi đưa cho model.

## 3. Tích hợp MCP client/server an toàn

1. Chỉ cài/chạy server có source và dependency đáng tin.
2. Kiểm tra mọi tool được khai báo, argument schema, resource và prompt.
3. Cấu hình credential với quyền tối thiểu cần thiết.
4. Ưu tiên capability read-only trong giai đoạn phát triển đầu.
5. Validate argument và resource ownership tại server, không chỉ trong prompt.
6. Thêm timeout, rate limit, audit record và human approval cho side effect nhạy cảm.
7. Thử request trái quyền, argument sai định dạng, server lỗi và tool schema thay đổi.

Transport và authentication thay đổi theo version protocol. Trước khi deploy, kiểm tra specification/SDK hiện hành. Coi output từ remote server là input không tin cậy.

## 4. So MCP với skill và tool thông thường

In-process function có thể là lựa chọn đơn giản nhất khi application kiểm soát cả hai phía. MCP hữu ích khi capability cần được khám phá và tái sử dụng giữa host/server tương thích. Skill dạy model một quy trình; MCP tool cung cấp capability có thể gọi; resource cung cấp context. Một agent có thể dùng cả ba, với permission rule tách biệt.

## Thực hành

Thiết kế MCP server read-only để tra tài liệu sản phẩm công khai. Định nghĩa một search tool, một document resource pattern, metadata trả về, hạn chế truy cập, timeout behavior và cách phát hiện server trả error hoặc nội dung prompt injection.

## Lỗi thường gặp

- Coi server nào sẵn có cũng đáng tin chỉ vì protocol chuẩn hóa.
- Cấp quyền database/filesystem rộng cho tool server.
- Nhầm MCP resource/prompt với executable tool do model gọi.
- Nghĩ server-side validation thay thế identity/authorization check phía client.
