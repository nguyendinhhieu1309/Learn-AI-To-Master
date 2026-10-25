# Tree, Binary Search Tree và Heap

[Xem bài gốc bằng tiếng Anh](../../../../curriculum/02-python/11-trees-bst-and-heaps.md)

Tree biểu diễn dữ liệu phân cấp. Binary Search Tree bổ sung quy tắc thứ tự; heap là cấu trúc dạng tree khác, được thiết kế để lấy nhanh giá trị cực trị.

## 1. Thuật ngữ về tree

Tree có node được nối bằng edge. Root là node trên cùng; các node con đi xuống từ một node; leaf không có node con. Depth của một node là khoảng cách từ root. Height là đường đi đi xuống dài nhất tới leaf.

Binary tree có tối đa hai node con mỗi node. Binary Search Tree (BST) duy trì quy tắc thứ tự như:

- giá trị trong left subtree nhỏ hơn node;
- giá trị trong right subtree lớn hơn node;
- quy tắc xử lý giá trị trùng được định nghĩa tường minh.

## 2. Tìm và chèn vào BST

~~~python
from __future__ import annotations

class TreeNode:
    def __init__(self, value: int) -> None:
        self.value = value
        self.left: TreeNode | None = None
        self.right: TreeNode | None = None

def insert(root: TreeNode | None, value: int) -> TreeNode:
    if root is None:
        return TreeNode(value)
    if value < root.value:
        root.left = insert(root.left, value)
    elif value > root.value:
        root.right = insert(root.right, value)
    return root

def contains(root: TreeNode | None, target: int) -> bool:
    current = root
    while current is not None:
        if target == current.value:
            return True
        current = current.left if target < current.value else current.right
    return False
~~~

Ví dụ này bỏ qua giá trị trùng. Mỗi thao tác tốn O(h), với h là chiều cao tree. BST cân bằng có h=O(log n); BST lệch có thể có h=O(n).

## 3. Duyệt tree

In-order traversal thăm left subtree, node, rồi right subtree. Với BST, cách duyệt này tạo ra các giá trị theo thứ tự tăng dần.

~~~python
def inorder(root: TreeNode | None):
    if root is None:
        return
    yield from inorder(root.left)
    yield root.value
    yield from inorder(root.right)
~~~

Pre-order thăm node trước các node con; post-order thăm sau các node con. Chúng hữu ích khi sao chép, serialize hoặc xóa tree. Mỗi traversal thăm n node, nên thời gian là O(n); call stack đệ quy là O(h).

## 4. Xóa khỏi BST

Có ba trường hợp cấu trúc:

1. Leaf: xóa node.
2. Có một node con: nối parent trực tiếp với node con.
3. Có hai node con: thay giá trị bằng in-order successor (giá trị nhỏ nhất trong right subtree) hoặc predecessor, rồi xóa node thay thế ở vị trí ban đầu.

Xóa có complexity O(h), với lưu ý về cân bằng giống thao tác tìm kiếm. BST cơ bản không tự giữ cân bằng.

## 5. Tree cân bằng

AVL tree và red-black tree dùng rotation sau các lần cập nhật để giữ cân bằng. Chúng duy trì height O(log n), nên search, insert và delete có complexity O(log n). Invariant và việc cân bằng lại phức tạp hơn; khi production cần ordered map, hãy dùng implementation thư viện đã được kiểm thử.

## 6. Heap và priority queue

Min-heap giữ giá trị nhỏ nhất ở root, nhưng những giá trị còn lại không được sắp xếp hoàn toàn. Trong Python, heapq cung cấp min-heap:

~~~python
import heapq

priorities = [7, 2, 5]
heapq.heapify(priorities)          # O(n)
heapq.heappush(priorities, 1)      # O(log n)
smallest = heapq.heappop(priorities)  # O(log n)
~~~

Dùng heap cho scheduler, xử lý top-k và tìm đường đi ngắn nhất trên graph. Với max-heap, có thể lưu priority số âm hoặc dùng tie-breaker tường minh trong tuple.

## Bài luyện tập kèm đáp án

1. Vì sao BST search có thể suy giảm thành O(n)?
2. Traversal nào trả về giá trị BST theo thứ tự tăng dần?
3. Heap có giữ mọi phần tử luôn được sắp xếp không?
4. Ba trường hợp xóa node trong BST là gì?

### Đáp án

1. Thứ tự insert có thể làm tree thành một chuỗi, khiến height bằng n.
2. In-order traversal.
3. Không; trong min-heap chỉ có root được đảm bảo là giá trị nhỏ nhất.
4. Leaf, có một node con và có hai node con.
