---
sr-due: 2025-06-18
sr-interval: 390
sr-ease: 204
reviewed: 2023-07-20
---

#dsa #graph

A [[Graph]] can sometimes be represented as matrix. Similarly to [[Tree traversal]] it is possible to perform depth first search and Breadth first search. It follows the same logic as [[Adjacency list traversal]] bit there are more edges cases to watch out for.

## Depth first search

Example problem: count the number of paths from top left to bottom right (must only have zeros)

```python
def dfs(grid, i, j, visit = None):
    if not visit:
        visit = set()
    if (i,j) in visit or i < 0 or j < 0 or i >= len(grid) or j >= len(grid[0]) or grid[i][j] == 1:
        return 0
    if i == len(grid) - 1 and j == len(grid[0])-1:
        return 1
    visit.add((i,j))
    tot = 0
    for r,c in [(i,j+1), (i,j-1), (i+1,j), (i-1,j)]:
        tot += dfs(grid, r, c, visit)
    visit.remove((i,j))
    return tot
dfs(grid,0,0)
```

Time complexity: O(4^(n.m))

## Breadth first search

Example problem: find the shortest path from top left to bottom right (must only have zeros)

````python
def bfs(grid):
    q = deque()
    visit = set()
    tot = 0
    q.append((0,0))
    visit.add((0,0))
    while q:
        for _ in range(len(q)):
          i,j = q.popleft()
            if i == len(grid) - 1 and j == len(grid[0]) -1:
                return tot
            for r,c in [(i,j+1), (i,j-1), (i+1,j), (i-1,j)]:
                if (r,c) not in visit and r >=0 and c >=0 and r < len(grid) and c < len(grid[0]) and grid[r][c]  == 0:
                    q.append((r,c))
                    visit.append((r,c))
        tot += 1
    return -1 # not reachable```
````

Time complexity: O(n.m)
