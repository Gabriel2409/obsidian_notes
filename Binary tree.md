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
