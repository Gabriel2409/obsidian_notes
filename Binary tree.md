---
sr-due: 2026-03-18
sr-interval: 659
sr-ease: 289
reviewed: 2023-08-02
---

#dsa #tree

## Definition

A Binary tree is a [[Tree]] where each node has at most two children

```python
@dataclass
class TreeNode:
    val:int
    left: Optional['TreeNode']
    right: Optional['TreeNode']
```

## Special trees

- **Full** Binary Tree: Each node has either 0 or 2 children. Can be skewed.
- **Balanced** Binary Tree: Each node left and right child have a height difference of at most 1 (balance of 1)
- **Perfect** Binary Tree: Each level is complete.
- **Complete** Binary Tree: Same as perfect but last level can be incomplete. Last level must be filled from left to right.

Note: A complete binary tree can easily be represented as an array:
```
        1
      /   \
     7     9
    / \   /  \
   4   7 6    1
  /
 3
can also be represented as
[1,7,9,4,7,6,1,3]

Index of first leaf node: len(arr)//2
Index of children of a node of index i: 2*i + 1 and 2*i + 2 (if they exist)
Index of parent of node i > 0:  (i+1)//2 - 1
```

## Height

Perfect, Complete and balanced Binary trees have a height of log n

Complete (and a fortiori perfect) are balanced but not the other way around

Example of balanced but not complete

```
    1
  /   \
 2     3
  \   /  \
   5 6    7
         /
        8
```
