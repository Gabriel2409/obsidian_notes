---
sr-due: 2023-07-06
sr-interval: 43
sr-ease: 269
reviewed: 2023-07-06
---

#dsa #tree

## Definition

A Binary Search Tree (BST) is a [[Binary tree]] where, for each node, the left child
and its descendants have a lower value AND the right child and its descendants
have a greater value. There are no duplicate values in a BST

```
      5
     / \
    2   6
   / \
  1   4
     /
    3
```

## Lookup

To find if an element exists, repeat:

- Start from root
- if element > root, go right, if element < root, go left

```python
def search(root, target):
    if not root:
        return False

    if target > root.val:
	    return search(root.right, val)
	elif target < root.val:
        return search(root.left, val)
    else:
        return True
```

- Average time complexity: O(logn) if the Tree is balanced
- Worst time complexity: O(n) if Tree is completely skewed

## Build from a sorted array (balanced)

In a sorted array, [[Binary search]] allows a lookup in O(logn) Time.
Similarly it is possible to build a balanced BST following a similar logic:

- Start with middle element as root
- set left child as middle element of left part and right child as middle element of right part
- repeat on left and right child

```python
def build(arr: list[int]):
    l = 0
    r = len(arr)
    return helper(arr, l, r)

def helper(arr, l, r):
    if l == r:
        return None
    else:
        mid = (l + r) // 2
        node = TreeNode(arr[mid])
        node.left = helper(arr, l, mid)
        node.right = helper(arr, mid + 1, r)
    return node
```

Starting r at len(arr) instead of the last index allows to have the extra element in the left subtree when the nb of elements in the array are even.
It also behaves similarly as passing arr[l:r]
This is also why we return None when l == r.

## Insertion

Insertion works like look up but we progress until we reach a node that does not have the needed child and then add the value

```python
# val assumed not in tree
def insert(root, val):
    if not root:
        return TreeNode(val)
    if val > root.val:
        root.right = insert(root.right, val)
    elif val < root.val:
        root.left = insert(root.left, val)
    return root
```

Note: This version of insertion does not maintain the balance of a balanced BST

## Deletion

- Scenario 1: node to delete has 0 or 1 child => Replace it with its child (or none if no child)
- Scenario 2: node has 2 children =>
  - Go to right child then to the left until you reach a node without left child (= final node)
  - Replace initial node value with value of final node (which basically holds the smallest value above the one we want to delete)
  - Apply the remove algorithm to the right child of the initial node with the final value. It will remove the now obsolete final node using scenario 1 (easy deletion) because the final node has at most one child

```python
# val assumed to be in tree
def get_min_node(root):
    while root.left:
        root = root.left
    return root

def remove(root, val):
    if val > root.val:
        root.right = remove(root.right, val)
    elif val < root.val:
        root.left = remove(root.left, val)

    else:
        if root.left is None:
            return root.right
        elif root.right is None:
            return root.left
        else:
            min_node = get_min_node(root.right)
            root.val = min_node.val
            root.right = remove(root.right, min_node.val)
    return root
```

Note: This version of deletion does not maintain the balance of a balanced BST
