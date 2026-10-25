# Linked list, stack và queue

[Xem bài gốc bằng tiếng Anh](../../../../curriculum/02-python/09-linked-lists-stacks-and-queues.md)

Các cấu trúc này giúp hiểu liên kết kiểu pointer và quy tắc thứ tự. Trong Python hằng ngày, list và deque thường là lựa chọn thực tế; tự cài đặt giúp hiểu rõ các đánh đổi.

## 1. Singly linked list

Mỗi node giữ một value và liên kết đến node tiếp theo. List lưu pointer đến head; ví dụ này cũng lưu pointer đến tail.

~~~python
from __future__ import annotations
from dataclasses import dataclass
from collections.abc import Iterator

@dataclass
class Node:
    value: int
    next: Node | None = None

class LinkedList:
    def __init__(self) -> None:
        self.head: Node | None = None
        self.tail: Node | None = None

    def prepend(self, value: int) -> None:
        node = Node(value, self.head)
        self.head = node
        if self.tail is None:
            self.tail = node

    def append(self, value: int) -> None:
        node = Node(value)
        if self.tail is None:
            self.head = self.tail = node
        else:
            self.tail.next = node
            self.tail = node

    def find(self, value: int) -> Node | None:
        current = self.head
        while current is not None:
            if current.value == value:
                return current
            current = current.next
        return None

    def delete_first(self, value: int) -> bool:
        previous = None
        current = self.head
        while current is not None:
            if current.value == value:
                if previous is None:
                    self.head = current.next
                else:
                    previous.next = current.next
                if current is self.tail:
                    self.tail = previous
                return True
            previous, current = current, current.next
        return False

    def __iter__(self) -> Iterator[int]:
        current = self.head
        while current is not None:
            yield current.value
            current = current.next
~~~

Khi có pointer head và tail, prepend và append tốn O(1). Tìm kiếm và xóa theo value tốn O(n); chỉ có thể xóa node đã biết trong O(1) nếu có thông tin về node đứng trước. Truy cập phần tử thứ k tốn O(n), khác với truy cập index O(1) ở list.

## 2. Doubly linked list

Node của doubly linked list cũng lưu liên kết đến node trước. Nhờ vậy có thể duyệt theo cả hai hướng và xóa node đã biết trong O(1), nhưng cần thêm bộ nhớ và phải cập nhật chính xác liên kết của hai node lân cận.

Linked list có thể hữu ích khi thường xuyên chèn hoặc xóa node sau khi đã tìm được vị trí. Bản thân việc tìm vị trí có thể vẫn tốn O(n). Linked list không tự nhiên nhanh hơn Python list; array có memory locality tốt và được tối ưu rất cao.

## 3. Stack: vào sau, ra trước

Stack lấy ra phần tử được thêm gần nhất. Dùng list với append và pop:

~~~python
stack = []
stack.append("open")
stack.append("close")
latest = stack.pop()
~~~

Push và pop ở cuối có amortized complexity O(1). Stack mô hình hóa lịch sử undo, lời gọi lồng nhau và duyệt depth-first.

## 4. Queue: vào trước, ra trước

Queue lấy ra phần tử được thêm sớm nhất. Dùng deque:

~~~python
from collections import deque

queue = deque()
queue.append("first")
queue.append("second")
next_item = queue.popleft()
~~~

Enqueue và dequeue ở hai đầu có complexity O(1). Tránh list.pop(0), vì thao tác đó tốn O(n).

## Bài luyện tập kèm đáp án

1. Vì sao append trong linked list có thể là O(1) khi có tail pointer nhưng O(n) khi không có?
2. Vì sao binary search không tìm được vị trí giữa linked list một cách hiệu quả?
3. Cấu trúc nào phù hợp với thao tác undo? Cấu trúc nào phù hợp với hàng đợi in?

### Đáp án

1. Có tail pointer thì đã biết node cuối; nếu không có, phải duyệt để tìm.
2. Linked list không thể nhảy đến index bất kỳ trong O(1).
3. Stack cho undo; queue cho hàng đợi.
