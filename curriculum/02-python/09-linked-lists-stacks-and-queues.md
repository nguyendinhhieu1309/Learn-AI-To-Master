# Linked Lists, Stacks, and Queues

These structures teach pointer-like links and ordering rules. In everyday Python, list and deque are usually the practical choice; implementing the structures is useful for understanding trade-offs.

## 1. Singly linked list

Each node holds a value and a link to the next node. The list keeps a head pointer; this version also keeps a tail pointer.

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

With a head and tail pointer, prepend and append are O(1). Search and delete-by-value are O(n); delete by known node can be O(1) only if predecessor information is available. Accessing the kth item is O(n), unlike a list's O(1) indexing.

## 2. Doubly linked list

A doubly linked node also stores a previous link. This allows traversal in both directions and removal of a known node in O(1), but needs more memory and careful updates to both neighbors.

Linked lists can be useful when nodes are frequently inserted or removed after locating a position. Finding that position may still cost O(n). They are not automatically faster than Python lists; arrays have good memory locality and are highly optimized.

## 3. Stack: last in, first out

A stack removes the most recently added item. Use a list with append and pop:

~~~python
stack = []
stack.append("open")
stack.append("close")
latest = stack.pop()
~~~

Push and pop at the end are amortized O(1). Stacks model undo history, nested calls, and depth-first traversal.

## 4. Queue: first in, first out

A queue removes the earliest added item. Use deque:

~~~python
from collections import deque

queue = deque()
queue.append("first")
queue.append("second")
next_item = queue.popleft()
~~~

Enqueue and dequeue at the ends are O(1). Avoid list.pop(0), which is O(n).

## Practice with answers

1. Why can a linked list append be O(1) with a tail pointer but O(n) without one?
2. Why cannot binary search efficiently locate the middle of a linked list?
3. Which structure fits undo operations? Which fits a printer waiting line?

### Answers

1. With a tail pointer, the last node is already known; without it, traversal must find it.
2. A linked list cannot jump to an arbitrary index in O(1).
3. Stack for undo; queue for the waiting line.
