#dsa

Not to be confused with stack in the context of [[Memory Allocation - Stack and heap|memory allocation]] although they share the LIFO principle

The stack data structure is a linear data structure that follows the LIFO principle. In a stack, elements are added and removed from the top, and the most recently added item is the first to be removed.

In Python, you can use a list to implement a stack, and the list provides methods like `append` (push), `pop`, and indexing (to read the latest element) that make it convenient to work with a stack-like data structure.

In a stack, pushing, popping and reading latest element is O(1)

```python

stack = []

stack.append(1)
stack.append(2)
stack.append(3)
stack.pop() # 3
```

