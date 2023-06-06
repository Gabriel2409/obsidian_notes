---
sr-due: 2023-07-26
sr-interval: 55
sr-ease: 250
---

#dsa #graph

## Definition

A linked list is **a special type of [[Graph]] where the nodes and edges form a chain-like structure**.
- The nodes at the end contain only one edge while the intermediate nodes contain two
- The list can be doubly linked (pointers in each direction)

```python
@dataclass
class Node:
	val: int # or anything
	next: Optional['Node'] # pointer to next node,
	prev: Optional['Node'] # pointer to prev node, only in doubly linked list
```

## Basic operations

### Lookup

To find a node with a given value, you need to traverse the whole list
```python
def lookup(root, val):
	while root:
		if root.val == val:
			return True
		root = root.next
	return False
```
- Time complexity: O(n)

### Insertion and deletion

- Insertion and deletion between two nodes consists in updating the pointers to point to the new node.
- Time complexity: O(1) provided we have the reference of the nodes where to perform insert / delete

```python
# Insertion
next_node = root.next
root.next = new_node
new_node.next = next_node
```
```python
# Deletion
root.next = root.next.next
```

## Advanced operations

- [[Fast and slow pointer (linked list)]]