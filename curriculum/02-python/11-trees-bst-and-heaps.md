# Trees, Binary Search Trees, and Heaps

Trees represent hierarchical data. A binary search tree adds an ordering rule; a heap is a different tree-shaped structure designed to retrieve an extreme value efficiently.

## 1. Tree vocabulary

A tree has nodes connected by edges. A root is the top node; children descend from a node; leaves have no children. The depth of a node is its distance from the root. The height is the longest downward path to a leaf.

A binary tree has at most two children per node. A binary search tree (BST) maintains an ordering rule such as:

- values in the left subtree are smaller than the node;
- values in the right subtree are larger;
- duplicate behavior is defined explicitly.

## 2. BST search and insertion

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

This example ignores duplicate values. Each operation takes O(h), where h is tree height. A balanced BST has h=O(log n); a skewed BST can have h=O(n).

## 3. Traversal

In-order traversal visits left subtree, node, then right subtree. For a BST, this produces values in sorted order.

~~~python
def inorder(root: TreeNode | None):
    if root is None:
        return
    yield from inorder(root.left)
    yield root.value
    yield from inorder(root.right)
~~~

Pre-order visits node before children; post-order visits after children. These are useful for copying, serializing, or deleting a tree. Every traversal visits n nodes, so time is O(n); recursion stack is O(h).

## 4. BST deletion

There are three structural cases:

1. Leaf: remove the node.
2. One child: connect its parent directly to its child.
3. Two children: replace its value with its in-order successor (smallest value in the right subtree) or predecessor, then delete that replacement node.

Deletion is O(h), subject to the same balancing caveat as search. A plain BST does not automatically stay balanced.

## 5. Balanced trees

AVL trees and red-black trees maintain balance using rotations after updates. They keep height O(log n), making search, insertion, and deletion O(log n). Their invariants and rebalancing are more complex; use a tested library implementation when production requirements need ordered map operations.

## 6. Heap and priority queue

A min-heap keeps the smallest value at the root, but the rest of the values are not fully sorted. In Python, heapq provides a min-heap:

~~~python
import heapq

priorities = [7, 2, 5]
heapq.heapify(priorities)          # O(n)
heapq.heappush(priorities, 1)      # O(log n)
smallest = heapq.heappop(priorities)  # O(log n)
~~~

Use a heap for schedulers, top-k processing, and graph shortest paths. For a max-heap, store negated numeric priorities or use an explicit tie-breaker in tuples.

## Practice with answers

1. Why can a BST search degrade to O(n)?
2. Which traversal returns BST values in ascending order?
3. Does a heap keep all elements sorted?
4. What are the three BST deletion cases?

### Answers

1. Insert order can make the tree a chain, so height becomes n.
2. In-order traversal.
3. No; only the root is guaranteed to be the minimum in a min-heap.
4. Leaf, one child, and two children.
