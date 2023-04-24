#dsa #tree

## Definition

Segment trees allow to easily calculate the sum over a range of values.
In a standard array, updating a value is O(1) but calculating the sum of values over
a given range is O(n).
Segment tress make update and sum over range calculation both O(logn)

```python
class SegmentTree:
    def __init__(self, total, L, R):
        self.sum = total # sum of the array
        self.left = None # pointer to left child
        self.right = None # pointer to right child
        self.L = L # left index
        self.R = R # right index
```

```text
    arr = [5,2,6]
             13[0,2]
            /    \
        7[0,1]   6[2,2]
        /   \
    5[0,0]   2[1,1]
```

## Building a segment trees from an array

- For each node, divide the remaining range by 2 to create the children.
  Then propagate up the value to get the sum

```python
def build(arr, L, R):
    if L == R:
        return SegmentTree(arr[L], L, L)
    node = SegmentTree(0, L, R)
    mid = (L + R) // 2
    node.left = build(arr, L, mid)
    node.right = build(arr, mid + 1, R)
    node.sum = node.left.sum + node.right.sum
    return node
```

Time complexity: O(n): go through all element of the array

## Update

- Navigate in the tree until we find the correct index, update the sum then propagate up

```python
def update(root, index, newval):
    if root.L == root.R:
        root.sum = newval
        return
    mid = (root.L + root.R) // 2
    if index > mid:
        update(root.right, index, newval)
    else:
        update(root.left, index, newval)
    root.sum = 0
    if root.right:
        root.sum += root.right.sum
    if root.left:
        root.sum += root.left.sum
```

Time complexity: O(logn): go down the height of the tree which is logn

## Get range sum

- Combines values from relevant nodes

```python

def range_query(root, L, R):

    if root.L  == L and root.R == R:
        return root.sum

    mid = (root.L + root.R)  // 2
    if R < mid:
        return range_query(root.left, L, R)
    elif L > mid:
        return range_query(root.right, L, R)
    else:
        return range_query(root.left, L, mid) + range_query(root.right, mid + 1, R)
```

Time complexity: O(logn): Indeed the last case is guaranted to happen at most once.
