# Lập trình hướng đối tượng trong Python

[Xem bài gốc bằng tiếng Anh](../../../../curriculum/02-python/06-object-oriented-programming.md)

Lập trình hướng đối tượng (Object-Oriented Programming, OOP) gom dữ liệu cùng hành vi thao tác lên dữ liệu đó. OOP có thể mô hình hóa các thực thể với quy tắc ổn định, nhưng với transformation đơn giản, function và dictionary thường rõ ràng hơn.

## Mục tiêu học tập

- Giải thích class, instance, attribute và method.
- Dùng constructor, instance/class/static method, property và special method.
- Áp dụng encapsulation, inheritance và polymorphism.
- Chọn composition thay cho inheritance khi các thành phần có thể thay đổi độc lập.

## 1. Class và instance

Class mô tả một loại object. Instance là một object cụ thể được tạo từ class đó. Instance attribute lưu trạng thái riêng của từng object.

~~~python
class Sensor:
    def __init__(self, sensor_id: str, unit: str) -> None:
        self.sensor_id = sensor_id
        self.unit = unit

    def describe(self) -> str:
        return f"{self.sensor_id} measures {self.unit}"

temperature_sensor = Sensor("temp-01", "degrees C")
print(temperature_sensor.describe())
~~~

Initializer nhận instance mới dưới tên self. Mỗi instance có sensor_id và unit riêng. Tránh dùng class attribute có thể thay đổi nếu mỗi instance cần list hoặc dictionary riêng.

## 2. Instance method, class method và static method

Instance method nhận self và có thể đọc hoặc thay đổi state của object. Class method nhận cls và có thể tạo hoặc kiểm tra class. Static method không tự nhận cả hai; nó phù hợp với helper liên quan đến class nhưng không cần state của object.

~~~python
class Prediction:
    valid_labels = {"cat", "dog"}

    def __init__(self, label: str, score: float) -> None:
        self.label = label
        self.score = score

    @classmethod
    def from_probability(cls, label: str, score: float) -> "Prediction":
        return cls(label, score)

    @staticmethod
    def valid_score(score: float) -> bool:
        return 0.0 <= score <= 1.0
~~~

## 3. Encapsulation và property

Encapsulation giữ quy tắc của object gần với state của nó. Python dùng convention đặt tên thay vì field private tuyệt đối: dấu gạch dưới đầu tên có nghĩa là “nội bộ; dùng thận trọng”.

~~~python
class Account:
    def __init__(self, balance: float = 0.0) -> None:
        self._balance = balance

    @property
    def balance(self) -> float:
        return self._balance

    def deposit(self, amount: float) -> None:
        if amount <= 0:
            raise ValueError("deposit must be positive")
        self._balance += amount
~~~

Property cung cấp interface chỉ đọc; deposit thực thi quy tắc state. Không tạo property cho mọi field nếu attribute trực tiếp đã đủ.

## 4. Inheritance và polymorphism

Inheritance biểu diễn quan hệ “là một” thực sự và cho phép subtype chuyên biệt hóa hành vi. Dùng super để gọi implementation của parent.

~~~python
class ImageSensor(Sensor):
    def __init__(self, sensor_id: str, width: int, height: int) -> None:
        super().__init__(sensor_id, "pixels")
        self.width = width
        self.height = height

    def describe(self) -> str:
        return f"{self.sensor_id}: {self.width}x{self.height}"
~~~

Polymorphism nghĩa là các object khác nhau có thể hỗ trợ cùng một thao tác. Python thường dùng duck typing: nếu object cung cấp method cần thiết, caller có thể dùng nó mà không cần cùng một parent class.

Đừng xây chuỗi inheritance sâu chỉ để chia sẻ vài dòng code. Ưu tiên composition khi object “có một” thành phần:

~~~python
class Camera:
    def __init__(self, calibration) -> None:
        self.calibration = calibration
~~~

Camera có một calibration object; calibration không cần kế thừa từ Camera.

## 5. Special method và data class

Special method cho phép object do người dùng định nghĩa hoạt động với các phép toán của Python. Ví dụ thường gặp gồm __repr__ để debug, __str__ để hiển thị, __eq__ để so sánh bằng nhau và __len__ để lấy độ dài. Chỉ triển khai khi hành vi rõ ràng và nhất quán.

Với record thuần dữ liệu, data class có thể gọn hơn việc viết boilerplate:

~~~python
from dataclasses import dataclass

@dataclass
class BoundingBox:
    x_min: float
    y_min: float
    x_max: float
    y_max: float
~~~

Data class tự tạo các method như initializer và representation dễ đọc. Nó không tự kiểm tra tính hợp lệ của tọa độ.

## 6. Checklist thiết kế OOP

- Object này có state và hành vi có ý nghĩa cần đặt cạnh nhau không?
- Invariant có được kiểm tra ở ranh giới không?
- Class có dễ dùng hơn function cộng một record đơn giản không?
- Composition có tránh được quan hệ inheritance mong manh không?
- Side effect và việc sửa object có được thể hiện rõ không?

## Bài luyện tập kèm đáp án

1. Class khác instance thế nào?
2. Khi nào method cần nhận self?
3. Dấu gạch dưới đầu tên attribute truyền đạt điều gì?
4. Vì sao composition có thể tốt hơn inheritance?

### Đáp án

1. Class định nghĩa một loại object; instance là một object cụ thể.
2. Khi thao tác cần state của instance hoặc cần thay đổi state đó.
3. Attribute được xem là nội bộ theo convention.
4. Các thành phần thay đổi độc lập có thể được thay thế mà không tạo hệ thống subtype sâu.
