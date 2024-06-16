---
sr-due: 2025-09-28
sr-interval: 491
sr-ease: 246
reviewed: 2023-07-12
---

#dsa #graph

## Definition

Prim algorithm allows to find the [[Minimum spanning tree]] in a connected undirected [[Graph]] by using an approach similar to [[Adjacency list traversal]] (BFS). The difference is that we keep track of the weight and use a heap instead of a standard queue

## Implementation

### Building the adjacency list

```python
adj = defaultdict(list)
for s,d,w in edges:
    adj[s].append((d,w))
    adj[d].append((s,w))
```

### Getting the nb of nodes

```python
found = set()
for s,d,w in edges:
    found.add(s)
    found.ads(d)
n = len(found)
```

### Main algorithm

```python
def prim(src, adj, n):
    visit = set()
    final = []
    h = []
    for d, w in adj[src]:
        heapq.heappush(h, (w, src, d))
    visit.add(src)

    while len(visit) < n:
        weight, src, dest = heapq.heappop(h)
        if dest in visit:
            continue
        visit.add(dest)
        final.append((src, dest))
        for d, w in adj[dest]:
            if d not in visit:
                heapq.heappush(h, (w, dest, d))

    return final

```

- Time complexity: O(ElogV)
