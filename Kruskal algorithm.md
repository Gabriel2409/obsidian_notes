---
sr-due: 2025-10-26
sr-interval: 515
sr-ease: 250
reviewed: 2023-07-12
---

#dsa #graph

## Definition

Kruskal algorithm is an algorithm to find the [[Minimum spanning tree]] in a connected undirected [[Graph]] by using [[UnionFind]]. It can work directly with the edges of the graphs (no need to create an adjacency list)
The idea is to place all the edges in a priority queue (based on the weight), then go through each of them and attempt to union the corresponding source and destination. If the union is successful, it means the nodes were not connected before and we have successfully added an edge.

## Implementation

### Getting the nb of nodes

```python
found = set()
for s,d,w in edges:
    found.add(s)
    found.add(d)
n = len(found)
```

### Main algorithm

```python
def kruskal(edges, n):
	h = []
	uf = UnionFind(n)
	final = []

	for s, d, w in edges:
		heapq.heappush(h, (w, s, d))

	nb_edges = 0
	while nb_edges < n - 1:
		w,s,d = heapq.heappop(h)
		if uf.union(s, d):
			final.append([s, d])
			nb_edges += 1
	return final



```

- Time complexity: O(ElogV): go through each edge and pop the heap where there are at most `E == V^2` elements
