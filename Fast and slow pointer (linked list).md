---
sr-due: 2023-05-16
sr-interval: 1
sr-ease: 230
---

#dsa
## Definition

- Method of traversing the linked list by using a slow pointer (traversing nodes one by one) and a fast pointer (traversing nodes two by two)

## Examples

### Finding middle of linked list

- If a linked list has no cycle, the last node points to None
- Finding the middle is just a matter of looking at the position of the slow pointer when the fast pointer has finished

```text
# slow pointer on top, fast in bottom
# we stop when the fast pointer is at the end

|             |             |
  A->B->C  => A->B->C => A->B-<C    -> middle is B
|                |                |
-------------------------

|          |
  A->B  => A->B =>
|             |
-------------------------
```

```python
def get_middle(head):
	slow = head
	fast = head.next
	while fast and fast.next:
		slow = slow.next
		fast = fast.next.next
	return slow
```

### Detecting cycle

- Go through the list with fast and slow pointers
- If slow and fast point to the same node at any given time, it means there is a cycle.
- Note that the node they point to is not the start of the cycle !

```text
# fast and slow point to the same node, there is a cycle
|	              |                |
A->B->C->D     A->B->C->D    A->B->C->D
    ↖___/          ↖___/         ↖___/
   |                    |          |
```

```python
def has_cycle(head):
	slow = head
	fast = head.next
	while fast and fast.next:
		slow = slow.next
		fast = fast.next.next
		if slow == fast:
			return True
	return False
```

### Detecting starting element in a cycle

- Once the cycle was detected, create a new slow pointer at the beginning (before the root)
  and advance both slow pointers. They will meet on the cycle start

```text
# extend previous example, below is slow2 pointer
        |	              |       |
  A->B->C->D     A->B->C->D    A->B->C->D
      ↖___/          ↖___/         ↖___/
|                |                |
```

```python
def cycle_start(head):
	slow = head
	fast = head.next
	while fast and fast.next:
		slow = slow.next
		fast = fast.next.next
		if slow == fast:
			return break

	slow = slow.next
	slow2 = head
	while True:
		if slow == slow2:
			return slow
		else:
			slow = slow.next
			slow2 = slow2.next
```

Why it works:

- A = dist from head to cycle start
- B = dist from cycle start to intersection slow / fast
- C = cycle length
- slow length = A + B
- fast length = A + B + C
- fast length = 2 \* slow length => A = C - B
- Distance from slow to cycle start = C - B
- Distance from slow 2 to cycle start = A
