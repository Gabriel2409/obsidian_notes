---
sr-due: 2023-05-20
sr-interval: 4
sr-ease: 230
---

#dsa #graph

## Definition

Topological sort works on Directed Acyclic [[Graph]], and consists in sorting the nodes so that children are always after their parents. Note that the graph does not need to be entirely connected.



```
	A               G
   ↙ ↘              ↓
  B   C             H
  ↓   ↓
  D   E
   ↘ ↙
    F

valid sorts:
[A,B,C,D,E,F,G,H]
[A,G,B,D,C,E,F,H]
[A,C,E,B,G,H,D,F]
```

## Implementation

### Building the adjacency matrix

```python
adj = {}
for s,d in edges:
	if s not in adj:
		adj[s] = []
	if d not in adj:
		adj[d] = []
	adj[s].append(d)
```

### Main algorithm

- We use dfs to find the last children and then go back.
- We iterate on all the nodes to retrieve nodes that are not connected.
- Finally, we invert the resulting array

```python
def topological_sort(adj):
	final = []
	visit = set()
	for node in adj.keys():
		dfs(node, adj, final, visit)
	final = final.reverse()
	return final

def dfs(node, adj, final, visit):
	if node in visit:
		return
	visit.add(node)
	for dest in adj[node]:
		dfs(node, adj, final, visit)
	final.append(node)

```

### Variation with cycle detection

A sort can not occur if the graph is not acyclical. We can integrate cycle detection in the algorithm.
The idea is to return False when a node we already visited is encountered again.
To do so we maintain a path set, and follow the [[Backtracking]] logic

```python
def topological_sort(adj):
	final = []
	visit = set()
	path = set() # to track nodes already added
	for node in adj.keys():
		if not dfs(node, adj, final, visit, path): # if check
			return []
	final = final.reverse()
	return final

def dfs(node, adj, final, visit, path):
	if node in path: #
		return False #
	if node in visit:
		return True
	visit.add(node)
	path.add(node) #
	for dest in adj[node]:
		dfs(node)
	path.remove(node) #
	final.append(node)
```
