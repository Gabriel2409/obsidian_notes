---
sr-due: 2023-06-09
sr-interval: 32
sr-ease: 270
---

#dsa #tree

## Definition

A heap is a [[Tree]] based data structure that implement an abstract data type called the priority queue.
It satisfies the following two properties

1. Structure property: A binary heap is a complete [[Binary tree]]. Because of that, heap is often implemented as an array as it is easy to represent a complete binary tree as an array
2. Order property:
   1. min heap: each node is smaller than its children
   2. max heap: each node is larger than its children

Note: the abstract data type is called a priority queue because the heap will rearrange itself when adding and popping elements so that we always pop the
element with the highest priority first (min element for min heap and max element for maxheap) even if said element was not added first.

```
        1
      /   \
     2     6
    / \   /  \
   5   7 6    7
  /
 6

can also be represented as
[1,2,6,5,7,6,7,6]
```

Python implement the min heap with the heapq module.
If a max heap is needed, the most simple solution is to multiply all the priorities by -1 and use the python min heap.

- look up min: O(1)
- push: O(logn)
- pop: O(logn)

## Array implementation (min heap)

### Percolate (sift)

- Satisfying the order property for the whole heap is the same as satisfying it for non leaf nodes.
- In order for a given node to satisfy the order property, we can either
  - percolate down (sift down)
    - we suppose all the index **AFTER** the one we want to percolate already satisfy the heap property and we want the node at index i to satisfy it as well.
    - we compare the node with its children. If it is larger than the smallest of its children, we swap and repeat on the children node (which now contains the swapped value)
  - percolate up (sift up)
    - we suppose all the index **BEFORE** the one we want to percolate already satisfy the heap property and we want the node at index i to satisfy it as well.
    - we compare the node with its parent. If it is smaller than the parent, we swap and repeat on the parent node (which now contains the swapped value)

Note: percolating time complexity is the height of the tree, i.e O(logn)

```
Percolate down

            X
         /      \
        X        3
      /  \      / \
     Y   Y     2   7
    / \ / \   /
   Y  Y Y  Y 4

Here we want the node with the value 3 to satisfy the heap property.
All the subsequent nodes already satisfy it. Note: Y represent nodes that are not important here that already satisfy the heap property
The previous nodes (X) are not important. They may or may not satisfy the heap property

2 is smaller than 3 : swap
3 is not smaller than 4, stop
```

```
Percolate up

            1
         /      \
        Y        6
      /  \      / \
     X   X     4   5
    / \ / \   /
   X  X X  X 3

Here we want the node with the value 3 to satisfy the heap property.
All the previous nodes already satisfy it. Note: Y represent nodes that are not important here that already satisfy the heap property
The subsequent nodes (X) are not important. They may or may not satisfy the heap property
3 is smaller than 4: swap
3 is smaller than 6: swap
3 is not smaller than 1: stop

```

```python
def percolate_down(arr, i):
    """nodes at subsequent indexes are supposed to already satisfy the order property"""
    left = 2 * i + 1
    right = 2 * i + 2
    smallest = i
    if left < len(arr) and arr[left] < arr[smallest]:
        smallest = left
    if right < len(arr) and arr[right] < arr[smallest]:
        smallest = right
    if smallest != i:
        arr[smallest], arr[i] = arr[i], arr[smallest]
        percolate_down(arr, smallest)


def percolate_down_iterative(arr, i):
    while 2 * i + 1 <= len(arr):
        smallest = i
        left = 2 * i + 1
        right = 2 * i + 2
        if left < len(arr) and arr[left] < arr[smallest]:
            smallest = left
        if right < len(arr) and arr[right] < arr[smallest]:
            smallest = right
        if smallest != i:
            arr[smallest], arr[i] = arr[i], arr[smallest]
        else:
            break

def percolate_up(arr, i):
    """nodes at previous indexes are supposed to already satisfy the order property"""
    parent = (i + 1) // 2 - 1
    if parent >= 0 and arr[parent] > arr[i]:
        arr[parent], arr[i] = arr[i], arr[parent]
        percolate_up(arr, parent)


def percolate_up_iterative(arr, i):
    while i > 0:
        parent = (i + 1) // 2 - 1
        if arr[parent] > arr[i]:
            arr[parent], arr[i] = arr[i], arr[parent]
        else:
            break
```

### Building the heap

- Building the heap can be done by percolating down all the non leaf nodes in reverse order. Indeed, percolating down supposes that the subsequent nodes already satisfy the heap propery.
- Note: we don't start at the end of the array because leaf nodes already satisfy the heap property

```python
def heapify(arr):
    for i in range(len(arr) // 2 - 1, -1, -1):
        percolate_down(arr, i)
```

- Time complexity: O(n)
  - Intuition: the first node needs to call the percolation h times where h is the height of the tree but the more you go down the tree, the less operations you need
  - Proof: Count the number of nodes and the nb of operations at each level, sum, recognize derivation of geometric sum, take value when h -> infinite

$$
\begin{aligned}
nbOp &= (\frac{n}{4}* 1) + (\frac{n}{8}* 2) + (\frac{n}{16}* 3) + ... + 1 * h \\
	&= \sum_{k=1}^h \frac{n}{4}\frac{k}{2^{k-1}} = \sum_{k=1}^h \frac{n}{4}kx^{k-1} \;\; with \; x= \frac{1}{2} \\
	&= \frac{n}{4}\frac{d}{dx}(\sum_{k=1}^h x^{k})_{x=1/2} <  \frac{n}{4}\frac{d}{dx}(\frac{1}{1-x})_{x=1/2} \\
	&= \frac{n}{4}(\frac{1}{(1-x)^2})_{x=1/2} \\
	&= n
\end{aligned}


$$

### Get min

- Time complexity: O(1)
- Note: getting the second min element takes significantly longer, this is not a sorted array

```python
def get_min(arr):
	return arr[0]
```

### Popping the heap

- To pop the heap, first, swap the first and last element of the array, then pop the array.
- Rearrange the heap by percolating down the new first node
- Complexity: O(logn)

```python
def hpop(arr):
    arr[0], arr[-1] = arr[-1], arr[0]
    val = arr.pop()
    percolate_down(arr, 0)
    return val
```

### Pushing to the heap

- To push to the heap, push to the array, then percolate up the last element
- Complexity: O(logn)

```python
def hpush(arr, val):
    arr.append(val)
    percolate_up(arr, len(arr) - 1)
```

## Heap sort

- Necessitates a small modification to the percolate function so that it takes a max index
- Works best with a max heap (a min heap will give the result in reverse order)
- Complexity: O(nlogn): apply the percolate function (O(logn)) on all elements
- Steps:
  - percolate down first element
  - swap first element with last
  - repeat but exclude last element from percolation

```python
def heap_sort(arr):
    heapify(arr)
    for i in range(len(arr)):
		# percolate_down is the same function as above but we replace len(arr)
		# with max_index so that it does not disturb the elements that are already sorted
        percolate_down(arr, 0, max_index=len(arr) - i)
        arr[0], arr[len(arr) - 1 - i] = arr[len(arr) - 1 - i], arr[0]
	arr = reverse(arr) # not needed with a max heap
    return arr
```

## Example application
- [[MedianFinder (Two heaps)]]



