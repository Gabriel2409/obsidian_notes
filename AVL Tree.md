#dsa #tree #todo

## Definition

AVL trees are [[BST]] that maintain balance on insertion and deletion by rotating some nodes.
In order to easily know the balance of a node, we must keep track of the height of each node.

```python
class AvlTreeNode:
   def __init__(self, val):
      self.val = val
      self.height = 1  # diff with standard BST: keep track of the height
      self.left = None
      self.right = None

def get_height(root):
   if not root:
      return 0
   return root.height

def get_balance(root):
   return get_height(root.left) - get_height(root.right)
```

## Rotations

Rotations are operations that are used to make a node more balanced while keeping the BST properties.

Note: To know in which direction to rotate a node, if its balance is positive, left rotate.

### Simple rotations

- Left rotation : right child of root becomes left child of root.right AND left child of root.right becomes root
- Right rotation : left child of root becomes right child of root.left AND right child of root.left becomes root

```
     5             9
    / \           / \
   A   9         5   C
      / \       / \
     B   C     A   B

```

Once the rotation is done, we must update the height of the concerned nodes. In the
example above, only 5 and 9 have their height modified

```python
def left_rotate(root):
   new_root = root.right
   root.right = new_root.left
   new_root.left = root
   root.height = 1 + max(get_height(root.right), get_height(root.left))
   new_root.height = 1 + max(get_height(new_root.right), get_height(new_root.left))
   return new_root

def right_rotate(root):
   new_root = root.left
   root.left = new_root.right
   new_root.right = root
   root.height = 1 + max(get_height(root.right), get_height(root.left))
   new_root.height = 1 + max(get_height(new_root.right), get_height(new_root.left))
```

### Left-right and right-left

These cases occur when balance of child is opposite sign of balance of parent.

- Right Left rotate: apply right rotation on root.right then left rotation on root

```
  1         1             2
    \       ↳\↰          / \
     3        2         1  3
   ↱/↲         \
   2            3
```

- Left right rotate: apply left rotation on root.left and right rotation on root

```
        3         3         2
       /        ↱/↲        / \
      1         2         1  3
      ↳\↰      /
        2     1

Left right rotate: apply left rotation on root.left and right rotation on root
```

## Insertion and deletion

Insertion and deletion are basically identical to a BST but after inserting/deleting the value, we go up all the nodes until we find 1 with a balance >1 or < -1. We then apply a rebalancing algorithm

For rebalancing, not all the nodes need to be checked, only those that were traversed during the insertion/deletion.

Note: contrary to classic BST, insertion and deletion can modify the root of the Tree.

### Rebalancing on insertion

As insertion puts the node at the end of the Tree, we vcan use the value to know what type of rotation to perform.

- If balance > 1 (more nodes on the left)
  - if inserted_value > node.left.val, we are in the left-right case [<]
  - else a simple left rotation is enough [ / ]
- If balance < -1 (more nodes on the right )
  - if inserted_value < node.right.val, we que in the right-left case [>]
  - else a simple right rotation is enough [ \\ ]

```python
def insert(root, val):
   if not root:
      return AvlTreeNode(val)
   if val == root.val:
      raise ValueError("Value already in tree")
   elif val > root.val:
      root.right = insert(root.right, val)
   else:
      root.left = insert(root.left, val)

   # -- AVL tree specific code below
   root.height = 1 + max(get_height(root.left), get_height(root.right))

   balance = get_balance(root)

   if balance > 1:
      if val > root.left.val:
            root.left = left_rotate(root.left)
      return right_rotate(root)
   elif balance < -1:
      if val < root.right.val:
            root.right = right_rotate(root.right)
      return left_rotate(root)
   return root
```

### Rebalancing on deletion

On deletion we can not use the value to decide which rotation to perform.
Instead, once we get an unbalanced node, we look at the balance of its child to decide

- If balance > 1
  - If balance root.left >=0, a right rotation is enough [ / ]
  - else we must perform a left right rotation [<]
    If balance < -1
  - If balance root.right <=0, a left rotation is enough [ \\ ]
  - else we must perform a right left rotation [>]

```python

# same as BST
def get_min_node(root):
   while root.left:
      root = root.left
   return root

def delete(root, val):
   if not root:
      raise ValueError("Value not in tree")
   if val > root.val:
      root.right = delete(root.right, val)
   elif val < root.val:
      root.left = delete(root.left, val)
   else:
      if not root.right:
            return root.left
      elif not root.left:
            return root.right
      else:
            min_node = get_min_node(root.right)
            root.val = min_node.val
            delete(root.right, min_node.val)

   # -- AVL tree specific code below
   root.height = 1 + max(get_height(root.left), get_height(root.right))

   balance = get_balance(root)

   if balance > 1:
      # contrary to insertion, we look at the balance of the child instead of comparing
      # it to the inserted value.
      if get_balance(root.left) < 0:
            root.left = left_rotate(root.left)
      return right_rotate(root)
   elif balance < -1:
      if get_balance(root.right) > 0:
            root.right = right_rotate(root.right)
      return left_rotate(root)

   return root

```

## Building from sorted array

- Same as classic BST but keep track of the height

```python
def convert_sorted_arr_to_balanced_bst(arr):
   def dfs(arr, l, r):
      if l == r:
            return None

      mid = (l + r) // 2

      root = AvlTreeNode(arr[mid])
      root.left = dfs(arr, l, mid)
      root.right = dfs(arr, mid + 1, r)

      # keep track of the height
      root.height = 1 + max(get_height(root.left), get_height(root.right))
      return root

   return dfs(arr, 0, len(arr))
```
