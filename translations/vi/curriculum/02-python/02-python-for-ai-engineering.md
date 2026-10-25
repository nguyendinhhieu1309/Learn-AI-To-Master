# Python cho AI Engineering

[Xem bài gốc bằng tiếng Anh](../../../../curriculum/02-python/02-python-for-ai-engineering.md)

Công việc AI cần nhiều hơn cú pháp notebook. Bài này xây dựng thói quen đáng tin cậy cho code số, package có thể tái sử dụng và thí nghiệm.

## 1. Cấu trúc dữ liệu và trực giác về độ phức tạp

Dùng list cho chuỗi có thứ tự và thay đổi được, tuple cho record cố định, dictionary cho tra cứu theo key, set cho membership và tính duy nhất, generator cho duyệt lười. Chọn đúng cấu trúc dữ liệu thường quan trọng hơn micro-optimization.

Tra cứu phần tử trong dictionary và set trung bình có độ phức tạp constant time; quét list có độ phức tạp tuyến tính theo độ dài. Lặp lại việc kiểm tra membership trong list bên trong vòng lặp lớn có thể biến tác vụ tưởng đơn giản thành công việc bậc hai.

## 2. Function, type và contract

Chia transformation thành các bước nhỏ, tường minh. Function cần làm rõ input, output, đơn vị, cách xử lý giá trị thiếu và side effect. Type hint cải thiện hỗ trợ của editor và giúp review interface dễ hơn; bản thân chúng không kiểm tra dữ liệu ở runtime.

~~~python
from collections.abc import Sequence

def normalize_texts(texts: Sequence[str]) -> list[str]:
    """Strip surrounding whitespace without mutating the input."""
    return [text.strip() for text in texts]
~~~

Với input không hợp lệ, hãy raise exception rõ nghĩa thay vì trả về giá trị mặc định trông có vẻ hợp lý nhưng sai. Tránh except Exception quá rộng vì nó che mất bug.

## 3. Tư duy array trong NumPy

Vector hóa phép toán trên array thay vì viết vòng lặp Python cho từng phần tử. Xác định rõ dtype và shape:

~~~python
import numpy as np

features = np.asarray([[1.0, 2.0], [3.0, 4.0]], dtype=np.float32)
mean = features.mean(axis=0, keepdims=True)
scale = features.std(axis=0, keepdims=True).clip(min=1e-6)
standardized = (features - mean) / scale
assert standardized.shape == features.shape
~~~

Ở đây axis=0 tính thống kê theo từng cột. keepdims=True giữ shape tương thích với broadcasting. Khi slicing, phân biệt view với copy; sửa một view có thể làm thay đổi array ban đầu.

## 4. Notebook và module

Dùng notebook để khám phá và giải thích. Chuyển transformation, metric và code training đã ổn định thành module có thể import để tái sử dụng và review. Notebook cần chạy lại được từ kernel sạch, tránh hidden state và ghi nhận input cùng artifact đầu ra.

Ví dụ cấu trúc project:

~~~text
project/
  pyproject.toml
  README.md
  src/project_name/
    data.py
    features.py
    train.py
    evaluate.py
  configs/
  notebooks/
  tests/
~~~

## 5. Environment và dependency

Dùng virtual environment riêng cho mỗi project và khóa các dependency trực tiếp trong lock file để deployment có thể tái tạo. Ghi lại phiên bản Python và thư viện accelerator. Tách dependency runtime khỏi công cụ phát triển. Tránh commit credentials, cache cục bộ, dataset hoặc model artifact lớn ngoài ý muốn.

## 6. Thí nghiệm có thể tái tạo

Truyền configuration tường minh thay vì dựa vào global ẩn. Đặt random seed cho Python, NumPy và ML framework, đồng thời nhớ rằng GPU kernel vẫn có thể không deterministic. Lưu configuration, phiên bản dữ liệu, code revision, metric và checkpoint cùng nhau.

## 7. Logging và xử lý lỗi

Dùng structured log với run ID và tên stage. Ghi count, shape, thời lượng và phần tóm tắt an toàn; không dump dữ liệu riêng tư hay toàn bộ prompt người dùng. Xử lý riêng các lỗi dự kiến, bổ sung ngữ cảnh và để lỗi bất ngờ hiện traceback.

## 8. Bài tập

1. Một vòng lặp kiểm tra mỗi ID trong một triệu ID có xuất hiện trong list gồm một triệu ID hay không. Set có thể thay đổi độ phức tạp ra sao?
2. Vì sao axis=0 tính mean theo feature cho matrix có shape (rows, features)?
3. Ngoài random seed, thông tin nào cần có trong bản ghi thí nghiệm để tái tạo model?
4. Vì sao notebook chỉ chạy được sau khi chạy các cell trước theo đúng một thứ tự cố định lại nguy hiểm?
