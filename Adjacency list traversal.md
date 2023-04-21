#dsa

A [[Graph]] can sometimes be as an adjacency list. Similarly to [[Tree traversal]] it is possible to perform depth first search and Breadth first search. This is easier than [[Matrix traversal]] as there are less edges cases.

## Depth first search

Example problem: count the number of paths from a source node to a target node

```python
def dfs(src, target, visit = None):
    if not visit:
        visit = set()
    if src in visit:
        return 0
    if src == target:
        return 1
    visit.add(src)
    tot = 0
    for dest in adj[src]:
        tot += dfs(dest, target, visit)
    visit.remove(src)
    return tot

```

Time complexity: O(N^V) where N is the Average number of edges for a node

## Breadth first search

Example problem: find the shortest path from src to target

```python
def bfs(src, target):
    q = deque()
    visit = set()
    tot = 0
    q.append(src)
    visit.add(src)
    while q:
        for _ in range(len(q)):
            node = q.popleft()
            if node == target:
                return tot
            for dest in adj[node]:
                if dest not in visit:
                    q.append(dest)
                    visit.append(dest)
        tot += 1
    return -1 # not reachable
```

Time complexity: O(V+E)
