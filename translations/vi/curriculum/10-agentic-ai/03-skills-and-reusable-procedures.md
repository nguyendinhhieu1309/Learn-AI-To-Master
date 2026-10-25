# Agent Skill và Quy trình Tái sử dụng

## Mục tiêu học tập

Phân biệt skill với tool, memory và runtime; đóng gói quy trình lặp lại trong thư mục portable; review instruction/script trước khi bật.

## 1. Skill làm gì?

Skill đóng gói kiến thức theo tác vụ: khi nào dùng, instruction cần theo, ví dụ, edge case và tùy chọn reference file/script. Nó giúp model áp dụng quy trình nhất quán. Skill không tự đảm bảo output đúng, không tự cấp quyền và không thực thi tool nếu host/runtime xung quanh không hỗ trợ.

Có thể hình dung từng phần:

- **Skill:** instruction và tài liệu hỗ trợ cho tác vụ cụ thể.
- **Tool:** năng lực thực thi có input, output và permission.
- **Memory/state:** thông tin được giữ lại cho hội thoại, user hoặc workflow.
- **Workflow/graph:** luồng điều khiển tường minh giữa các bước và tool.
- **Harness:** runtime và policy cung cấp context, thực thi action, giữ state và quan sát kết quả.

## 2. Thư mục skill có thể mang đi

Agent Skills specification mô tả thư mục có file bắt buộc SKILL.md và các thư mục tùy chọn scripts/, references/, assets/. File SKILL.md bắt đầu bằng YAML frontmatter; name và description là trường bắt buộc. Tên dùng chữ thường, số và dấu gạch nối; description nêu skill làm gì và dùng khi nào.

~~~text
skills/
└── csv-quality-audit/
    ├── SKILL.md
    ├── references/
    │   └── quality-checks.md
    └── scripts/
        └── profile_csv.py
~~~

~~~markdown
---
name: csv-quality-audit
description: Profile a CSV schema, missing values, duplicates, and suspicious ranges. Use when auditing a new tabular dataset before modeling.
---

# CSV quality audit

1. Confirm the file's source, license, and row meaning.
2. Inspect its shape, columns, types, nulls, duplicates, and value ranges.
3. Report findings without deleting or rewriting the original data.
4. Ask before applying a cleaning rule that could remove valid observations.

See references/quality-checks.md for the full checklist.
~~~

Định dạng cho phép metadata tùy chọn như license/compatibility và trường thử nghiệm allowed-tools; mức hỗ trợ tùy host. Runtime permission policy phải được giữ ngoài nội dung skill.

## 3. Progressive disclosure

Description cần cụ thể để host quyết định skill có liên quan không. Đặt procedure chính trong SKILL.md; chuyển tài liệu dài sang file tham chiếu, code chi tiết sang script. Nhờ vậy host chỉ nạp skill khi hữu ích thay vì nhét mọi quy trình vào mọi prompt.

Đừng tạo skill cho một instruction dùng một lần. Skill phù hợp là công việc lặp, có phạm vi rõ, input, bước, artifact mong đợi và edge case phổ biến: kiểm toán dữ liệu, review paper, hiệu chuẩn camera, báo cáo thí nghiệm hoặc phân loại sự cố.

## 4. Làm procedure có thể kiểm chứng

Skill tốt định nghĩa input bắt buộc, thứ tự bước, output kỳ vọng, validation, điều kiện dừng/hỏi và failure case đã biết. Với skill tạo code hoặc phân tích, nêu cách kiểm tra kết quả. Skill cần nói khi evidence không đủ và cách báo uncertainty.

## 5. Security review

Skill có thể tham chiếu script/tool nên là input trong supply chain. Đọc instruction và kiểm tra script trước khi kích hoạt. Kiểm tra network access, file access, dependency, secret và side effect. Không tin skill chỉ vì format đẹp hoặc tải từ repository. Cấp tool được dẫn chiếu đúng mức quyền cần thiết.

## Thực hành

Viết skill “phân tích false negative của model”. Nêu khi nào áp dụng, file/input cần, error taxonomy theo bước, output format, privacy constraint và quy tắc ngăn agent thay đổi test set/model khi chưa được duyệt.

## Lỗi thường gặp

- Viết description mơ hồ khớp quá nhiều hoặc không khớp tác vụ nào.
- Đưa hàng trăm trang vào instruction chính thay vì nạp reference khi cần.
- Coi skill là sandbox, safety policy hoặc persistent memory.
- Chạy script đi kèm trước khi review permission và hành vi.
