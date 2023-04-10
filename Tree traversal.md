#dsa #tree #todo

## Definition
Traversal corresponds to the way to visit all the notes in a Tree.
Note focuses on [[Binary tree]].

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
Start on root. Recursively call dfs on left and then right.
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
### In order traversal
- Traversal order is left -> root -> right.
- Called in order because it visits all the nodes of a [[BST]] In ascending order.
- Can not be easily extended to non Binary trees as it would require to visit some of the children before the root and some of them after.
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
Idea is to use a queue to store all the children and then pop the queue in order

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
