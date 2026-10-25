# Python Production và Cấu trúc Project

## Mục tiêu học tập

Tổ chức AI project để đồng nghiệp cài đặt được, hiểu trách nhiệm từng phần và chạy cùng workflow mà không phụ thuộc state của notebook.

## 1. Từ notebook thành project

Notebook hữu ích cho exploration. Logic production nên nằm trong function/module có thể import để tái sử dụng từ command, API, scheduled job hoặc test. Giữ notebook cho thí nghiệm và báo cáo; chuyển function ổn định vào package.

~~~text
my-ai-project/
├── pyproject.toml       # package và cấu hình tool
├── README.md            # setup và cách dùng
├── src/my_ai_project/
│   ├── data.py
│   ├── features.py
│   ├── model.py
│   └── inference.py
├── tests/
└── notebooks/
~~~

Tách trách nhiệm: đọc data không nên âm thầm train model; prediction không nên phụ thuộc vào cell notebook đã chạy trước.

## 2. Environment và dependency

Dùng project environment như venv, Conda hoặc manager khác để tránh dependency conflict giữa các project. Ghi dependency trực tiếp và khóa resolved version cho deployment hoặc công việc cần tái lập. Pin mọi thứ mãi mãi có thể cản security fix; hãy cập nhật có chủ đích rồi chạy lại các bước kiểm tra liên quan.

Giữ configuration ngoài code: file path, model name, batch size, threshold và service endpoint nên là setting. Đặt credential trong secret manager hoặc injected environment; tuyệt đối không commit vào source control, notebook, log hay skill file.

## 3. Function, type và error

Ưu tiên function nhỏ với input/output rõ. Type hint mô tả cấu trúc mong đợi và hỗ trợ static tool; nó không thay runtime validation đối với input không tin cậy.

~~~python
from pathlib import Path

def load_text(path: Path, *, encoding: str = "utf-8") -> str:
    """Read a UTF-8 text file or raise a useful file/encoding error."""
    return path.read_text(encoding=encoding)
~~~

Raise error cụ thể khi caller có thể xử lý. Đừng biến mọi lỗi thành None, dataset rỗng hoặc câu trả lời model chắc chắn. Đưa đủ context để debug mà không lộ giá trị nhạy cảm.

## 4. Logging và configuration

Dùng structured log có timestamp, operation name, model/data version, request ID và duration. Tách metric khỏi log chi tiết. Mặc định không log password, API key, dữ liệu cá nhân thô hoặc toàn bộ prompt. Đọc setting tại ranh giới application rồi truyền giá trị cần thiết vào function một cách tường minh.

## 5. Reproducibility

Ghi code revision, data version, model version, parameter, environment, split definition và metric. Random seed chỉ tái lập lựa chọn giả ngẫu nhiên trong giới hạn; GPU kernel, package version và thực thi đồng thời vẫn có thể tạo khác biệt.

## Thực hành

Tách bước làm sạch data và prediction từ notebook thành function. Chạy trong environment mới. Đổi input path qua configuration, tạo lỗi thiếu file và kiểm tra thông báo giải thích đúng vấn đề mà không lộ dữ liệu riêng.

## Lỗi thường gặp

- Dựa vào thứ tự chạy notebook hoặc global variable ẩn.
- Đặt model, API, data và UI logic chung một file lớn.
- Cài dependency mà không ghi cách tạo environment.
- Để secret trong file .env vô tình bị commit hoặc trong notebook output.
