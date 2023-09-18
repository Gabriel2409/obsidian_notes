---
reviewed: 2023-07-05
---

#dsa #tree #todo

## Overview

- Generalisation of [[BST]] = self balancing tree which can handle large amount of data by sorting multiple keys per node

```python
@dataclass
class BTreeNode:
    leaf: boolean
    keys: List[int]
    children: List['BTreeNode']
```

## Properties

- All leaves are at the same level.
- B-Tree is defined by its minimum degree `t` which depends upon disk block size.
- Nodes must all have at least `t-1` and at most `2t-1` keys
- root can have less keys (at least `1` key)
- Number of children of a node is equal to the number of keys in it plus 1
- All keys of a node are sorted in increasing order. The child between two keys `k1` and `k2` contains all keys in the range from `k1` and `k2` (not included)
- B-Tree grows and shrinks from the root which is unlike Binary Search Tree. Binary Search Trees grow downward and also shrink from downward.
- Like other balanced Binary Search Trees, the time complexity to search, insert and delete is O(log n).

## Search

- searching an element is done similarly to what is done is a [[BST]]. The only difference is that you loop on the keys of each node

```python
def search(root, key):

    if root is None:
        return False

    i = 0
    while i < len(root.keys) and key > root.keys[i]:
        i += 1

    if i < len(root.keys) and key == root.keys[i]:
        return True

    if root.leaf:
        return False

    return search(root.children[i], key)

```

## Traversal

- traversal also follow a similar logic as binary trees : see [[Tree traversal]]. But there is no notion of inorder traversal

```python
def dfs(root:BTreeNode):
    final = []
    for key in root.keys:
        final.append(key)
    for child in root.children:
        final.extend(dfs(child))
    return final
    # for postorder, run second loop first
```

## Insert

- start from root and traverse it to find the leaf node where to insert new element
- insert element in the leaf node
- if leaf node is full, split it into two nodes and move median to parent node.
- id parent is full, recursively split parent until parent is not full
- if you reach the root, create a new root (height increases by 1)

```python
def insert(root, key):
    """inserts key in the b tree and returns the root"""
    if not root:
        return BTreeNode(keys=[key], leaf=True)

    if root.leaf:
        insert_into_leaf(root, key)
        return root

    else:
        i = 0
        while i < len(root.keys) and key > root.keys[i]:
            i += 1

        child = root.children[i]
        if len(child.keys) == 2 * t - 1:
            root = split_child(root, i)

        if key > root.keys[i]:
            i += 1

        root.children[i] = insert(root.children[i], key)
        return root

def insert_into_leaf(node, key):
    i = 0
    while i < len(node.keys) and key > node.keys[i]:
        i += 1
    node.keys.insert(i, key)

def split_child(parent, index):
    child = parent.children[index]
    new_child = BTreeNode(leaf=child.leaf)

    # Move the median key to the parent node
    median = child.keys[t - 1]
    parent.keys.insert(index, median)

    # Split the child's keys and children between child and new_child
    parent.children.insert(index + 1, new_child)
    new_child.keys = child.keys[t:]
    child.keys = child.keys[:t - 1]

    if not child.leaf:
        new_child.children = child.children[t:]
        child.children = child.children[:t]

    return parent
```

## Delete

- search for the key. If not found in the tree, stop
- if key is in leaf node, simply remove it
- if key not in leaf node:
  - if key has a successor / predecessor within the node, replace key with it
    then recursively delete successor / predecessor
  - if it does not have successor / predecessor, replace key with minimum from right
    subtree or max from left subtree and recursively delete max or min key from appropriate
    subtree
  - if it appears more than once, recursively delete it from appropriate subtree
- if node becomes underfull, redistribute key from immediate siblings or merge with sibling.
- continue restructuring until you reach a node that does not violate the min degree property
- if root becomes empty, update it to be its only child

```python
def delete(root, key):
    if not root:
        return root

    i = 0
    while i < len(root.keys) and key > root.keys[i]:
        i += 1

    if i < len(root.keys) and root.keys[i] == key:
        # Case 1: Key is in this node
        if root.leaf:
            # Case 1a: Key is in a leaf node
            root.keys.pop(i)
        else:
            # Case 1b: Key is in an internal node
            root.keys[i] = find_predecessor(root, i)
            root.children[i] = delete(root.children[i], root.keys[i])
    else:
        # Case 2: Key is not in this node
        if root.leaf:
            # Key is not in the tree
            return root

        if len(root.children[i].keys) < 2:
            # Case 2a: The child has fewer than t keys
            root = fix_underflow(root, i)

        root.children[i] = delete(root.children[i], key)

    return root

# Helper functions for delete operation
def find_predecessor(node, index):
    # Find the predecessor of the key at index in the node
    node = node.children[index]
    while not node.leaf:
        node = node.children[-1]
    return node.keys[-1]

def fix_underflow(node, index):
    # Fix underflow in the node by borrowing from siblings or merging
    if index > 0 and len(node.children[index - 1].keys) >= 2:
        # Borrow a key from the left sibling
        child = node.children[index]
        left_sibling = node.children[index - 1]
        child.keys.insert(0, node.keys[index - 1])
        node.keys[index - 1] = left_sibling.keys.pop()
        if not child.leaf:
            child.children.insert(0, left_sibling.children.pop())

    elif index < len(node.children) - 1 and len(node.children[index + 1].keys) >= 2:
        # Borrow a key from the right sibling
        child = node.children[index]
        right_sibling = node.children[index + 1]
        child.keys.append(node.keys[index])
        node.keys[index] = right_sibling.keys.pop(0)
        if not child.leaf:
            child.children.append(right_sibling.children.pop(0))

    else:
        # Merge with a sibling
        if index > 0:
            left_sibling = node.children[index - 1]
            merge_index = index - 1
        else:
            left_sibling = node.children[index + 1]
            merge_index = index

        merge_nodes(node, merge_index)

    return node

def merge_nodes(node, merge_index):
    # Merge the node with its sibling at merge_index
    child = node.children[merge_index]
    sibling = node.children[merge_index + 1]
    child.keys.append(node.keys[merge_index])
    child.keys.extend(sibling.keys)
    if not child.leaf:
        child.children.extend(sibling.children)
    node.keys.pop(merge_index)
    node.children.pop(merge_index + 1)
```
