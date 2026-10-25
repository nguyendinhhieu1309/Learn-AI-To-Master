# Ngôn ngữ Python cốt lõi

[Xem bài gốc bằng tiếng Anh](../../../../curriculum/02-python/04-python-core-language.md)

Bài này xây dựng mô hình tư duy về giá trị, tên, luồng điều khiển và các collection tích hợp của Python dùng trong data science và AI.

## Mục tiêu học tập

- Viết expression, nhánh điều kiện và vòng lặp.
- Chọn giữa string, list, tuple, set và dictionary.
- Giải thích indexing, slicing, mutability và reference.
- Dùng comprehension mà không giấu logic phức tạp.

## 1. Giá trị và kiểu dữ liệu

Tên biến tham chiếu đến một object. Phép gán gắn một tên với object; không nhất thiết tạo bản sao.

~~~python
age = 21
temperature = 22.5
name = "Linh"
is_ready = True
result = None
~~~

Các kiểu phổ biến gồm integer, floating-point number, string, boolean và None. Python có kiểu động (dynamically typed): một tên có thể được gán lại, nhưng các phép toán vẫn phải tuân theo quy tắc kiểu. Chuyển đổi ở ranh giới input bằng int, float hoặc str; chuyển đổi không hợp lệ sẽ raise ValueError.

## 2. Toán tử và phép so sánh

Toán tử số học gồm cộng, trừ, nhân, chia, chia lấy phần nguyên, chia lấy dư và lũy thừa. Phép so sánh trả về boolean. Dùng in để kiểm tra membership, == để so sánh giá trị và is để kiểm tra object identity.

Dùng is None để kiểm tra giá trị None:

~~~python
if label is None:
    print("label is missing")
~~~

Không dùng is để so sánh string hay số thông thường. Kết hợp điều kiện bằng and, or và not. Thêm ngoặc vào biểu thức so sánh hỗn hợp để dễ đọc.

## 3. String

String là chuỗi Unicode bất biến. Index bắt đầu từ 0; index âm đếm từ cuối. Slice lấy phần tử ở vị trí bắt đầu nhưng loại trừ vị trí kết thúc.

~~~python
word = "robotics"
print(word[0])      # r
print(word[-1])     # s
print(word[0:5])    # robot
print(word[::2])    # every second character
~~~

Các method hữu ích gồm strip, lower, split, replace và startswith. Dùng f-string để định dạng đầu ra:

~~~python
model_name = "baseline"
accuracy = 0.875
print(f"{model_name}: {accuracy:.1%}")
~~~

## 4. List và tuple

List có thứ tự và thay đổi được. Tuple có thứ tự và bất biến sau khi tạo. List có các method append, extend, insert, remove, pop, sort và copy.

~~~python
features = [0.2, 1.4, 3.0]
features.append(4.1)
first_two = features[:2]
coordinates = (12.5, 8.0)
x, y = coordinates
~~~

sorted(values) trả về list mới đã sắp xếp; values.sort() thay đổi list hiện tại. Tránh xóa phần tử khỏi list trong khi đang duyệt, vì vị trí các phần tử còn lại sẽ dịch chuyển.

## 5. Dictionary và set

Dictionary ánh xạ key có thể hash sang value. Set lưu các giá trị duy nhất có thể hash. Cả hai đều hữu ích cho tra cứu theo key và kiểm tra membership lặp lại.

~~~python
scores = {"image_a": 0.92, "image_b": 0.73}
scores["image_c"] = 0.88
print(scores.get("unknown", 0.0))

seen_ids = {"a17", "b29"}
print("a17" in seen_ids)
~~~

Dùng get khi key không tồn tại cần có giá trị mặc định rõ ràng. List và dictionary không thể làm key vì chúng có thể thay đổi và không hashable.

## 6. Nhánh, vòng lặp và comprehension

~~~python
for score in [0.72, 0.91, 0.64]:
    if score >= 0.8:
        print("passed")
    else:
        print("review")

positive = [value for value in values if value > 0]
~~~

Dùng for để xử lý collection hoặc một khoảng lặp đã biết. Dùng while khi việc lặp phụ thuộc vào điều kiện thay đổi. Đảm bảo mỗi while loop có thể đạt đến điều kiện dừng. Dùng vòng lặp thông thường nếu comprehension chứa nhiều điều kiện hoặc side effect.

## 7. Mutability, aliasing và copying

Hai tên có thể cùng tham chiếu đến một list có thể thay đổi:

~~~python
original = [1, 2]
alias = original
alias.append(3)
print(original)  # [1, 2, 3]
~~~

list.copy tạo shallow copy; các object lồng bên trong vẫn được dùng chung. Deep copy tốn kém hơn, chỉ dùng khi thực sự cần bản sao độc lập đệ quy.

## Bài luyện tập kèm đáp án kiểm tra

1. Value equality khác identity thế nào?
2. Vì sao thay đổi list thông qua alias lại ảnh hưởng đến tên ban đầu?
3. Cấu trúc nào biểu diễn tập nhãn duy nhất một cách tự nhiên?
4. [2, 5, 8, 11][1:3] trả về gì?

### Đáp án

1. Equality so sánh giá trị; identity kiểm tra các tên có tham chiếu cùng object không.
2. Cả hai tên tham chiếu cùng một list có thể thay đổi.
3. Set.
4. [5, 8]; index kết thúc bị loại trừ.
