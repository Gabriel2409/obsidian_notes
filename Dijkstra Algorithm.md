---
sr-due: 2023-06-01
sr-interval: 24
sr-ease: 264
---

#dsa #graph

## Definition

Bfs can be used to find the shortest path In a standard graph (see [[Matrix traversal]] and [[Adjacency list traversal]]).

Dijkstra allows to find the shortest path in a directed weighted [[Graph]].

Algorithm is very similar but we replace the basic queue with a min heap to account for the different weights.

## Implementation

### Building the adjacency list

```python
adj = defaultdict(list)
for s,d,w in edges:
    adj[s].append((d,w))
```

### Main algorithm

```python
def dijkstra(src, adj):
    h = []
    shortest = {}
    heap.heappush(h, (0, src))
    while h:
        weight, node = heap.heappop()
        if node in shortest:
            continue
        shortest[node] = weight
        for d, w in adj[node]:
            if d not in shortest:
                heap.heappush(h, (weight + w, d))
    return shortest
```

- Time complexity: O(ElogE)
- This algorithm only works if all weights are positive. If paths can be negative, we must use [[Bellman Ford Algorithm]]
