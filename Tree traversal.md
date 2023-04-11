#dsa #tree

## Definition

Traversal corresponds to the way to visit all the notes in a Tree.
This note focuses on [[Binary tree]].

## Depth first search (dfs)

Type of traversal where we explore each node as far as possible alongside a given branch before backtracking

### Pre order traversal

- Traversal order is root -> left -> right.
- Can be extended to non Binary trees (root then children)

```
     1
   /   \
  2     5
 / \   / \
3   4 6   7

```

#### Implementation

##### Recursive

Add root value then recursively call dfs on left and then recursively call dfs on right.

```python
def dfs(root:TreeNode):
    final = []
    final.append(root.val)
    if root.left:
        final.extend(dfs(root.left))
    if root.right:
        final.extend(dfs(root.right))
    return final
```

##### Iterative

1. Start with root in stack.
2. (Repeat) Pop stack, add current value then push right and then left in stack. Right is pushed
   first because it is accessed last.

```python
def dfs(root:TreeNode):
    final = []
    stack = [root]

    while stack:
        root = stack.pop()
        final.append(root.val)
        if root.right:
            stack.append(root.right)
        if root.left:
            stack.append(root.left)

    return final
```

### In order traversal

- Traversal order is left -> root -> right.
- Called in order because it visits all the nodes of a [[BST]] In ascending order.
- Can not be easily extended to non Binary trees as it would require to visit some
  of the children before the root and some of them after.

```
     4
   /   \
  2     6
 / \   / \
1   3 5   7
```

#### Implementation

##### Recursive

Recursively call dfs on left then go to root and then recursively call dfs on right.

```python
def dfs(root:TreeNode):
    final = []
    if root.left:
        final.extend(dfs(root.left))
    final.append(root.val)
    if root.right:
        final.extend(dfs(root.right))
    return final
```

##### Iterative

Repeat:

- start on root and add it to stack then go left until there is no left child
- pop the stack, get current value and set root to right child (which can be null)

```python
def dfs(root:TreeNode):
    final = []
    stack = []

    while root or stack:
        if root:
            stack.append(root)
            root = root.left
        else:
            root = stack.pop()
            final.append(root.val)
            root = root.right
    return final
```

### Post order traversal

- Traversal order is left -> right-> root
- Can be extended to non Binary trees (children then root)

```
     7
   /   \
  3     6
 / \   / \
1   2 4   5
```

#### Implementation

##### Recursive

Recursively call dfs on left then recursively call dfs on right then go to root

```python
def dfs(root:TreeNode):
    final = []
    if root.left:
        final.extend(dfs(root.left))
    if root.right:
        final.extend(dfs(root.right))
    final.append(root.val)
    return final
```

##### Iterative

2 stack solution:

- Create two stacks to track nodes and if they were already visited, initialised with root and False
- Each time you pop the stacks, check if the node was visited.
  - If yes, it means you already went right and you can safely add the current value.
  - If no, put back the element in the stack and add True to the visited stack. Then add
    right and left children to stack with visited set to False

```python
def dfs(root:TreeNode):
    stack = [root]
    visit = [False]
    final = []

    while stack:
        root = stack.pop()
        is_visited = visit.pop()
        if is_visited:
            final.append(root.val)
        else:
            stack.append(root)
            visit.append(True)
            if root.right:
                stack.append(root.right)
                visit.append(False)
            if root.left:
                stack.append(root.left)
                visit.append(False)

    return final

```

1 stack solution:

Repeat:

- start on root and add it to stack. If root has a right child, add it to stack
  then go left until there is no left child
- pop the stack:
  - If last element of stack is right child of current, put back current in
    stack and go right.
  - Else use current value and set root to null

```python
def dfs(root:TreeNode):
    final = []
    stack = []

    while root or stack:
        if root:
            if root.right:
                stack.append(root.right)
            stack.append(root)
            root = root.left
        else:
            root = stack.pop()
            if stack and root.right and stack[-1] == root.right:
                stack.pop()
                stack.append(root)
                root = root.right
            else:
                final.append(root.val)
                root = None
    return final
```

## Breadth first search (bfs)

Explores all nodes at a given depth before moving to the next depth level

### Level order traversal

- Can easily be extended to non Binary trees

```
     1
   /   \
  2     3
 / \   / \
4   5 6   7
```

#### Implementation

##### Iterative

Idea is to use a queue to store all the children and then pop the queue in order.
To track the level as well, a possibility is to add an extra for loop after the while
to pop just the right number of elements after going to the next level

```python
from collections import deque
def bfs(root):
    final = []
    q = deque()
    q.append(root)

    while q:
        root = q.popleft()
        final.append(root.val)
        if root.left:
            q.apend(root.left)
        if root.right:
            q.append(root.right)
    return final

def bfs_with_levels(root):
    final = []
    q = deque()
    q.append(root)

    while q:
        final.append([])
        # get out of for loop after popping the current level
        # other possibility is to track the current level in the queue
        for _ in range(len(q)):
            root = q.popleft()
            final[-1].append(root.val)
            if root.left:
                q.apend(root.left)
            if root.right:
                q.append(root.right)
    return final
```
