---
sr-due: 2023-05-10
sr-interval: 1
sr-ease: 210
---

#dsa #graph

## Definition

Kosaraju algorithm is an algorithm to detect [[Strongly connected components]] In a directed [[Graph]]

- Step 1: building the stack
  - Start on a random node and perform a post order dfs to get all the reachable nodes in a stack
  - Repeat operation on all nodes that were not reached to complete the stack
  - The output of this step is a stack containing all the nodes with children before parents
- Step 2: Finding the SCCs:
  - Pop the stack and perform a dfs on the resulting node but on the **REVERSED** graph to get a strongly connected component. Note that this dfs can be a preorder or à post order dfs, it does not matter
  - Keep popping the stack to get the other components (ignore the nodes that were already visited)

```text
Step 1

A → B     F
↑   ↓    ↗ ↘
D ← C → E ← G → H

Start on a random node, for ex E
dfs => Example stack = [H, G, F, E]
Go to a node not already visited, for ex B
stack = [A, D, B, C]
Final stack = [H, G, F, E, A, D, B, C]
```

```text
Step 2: dfs on reversed graph

A ← B     F
↓   ↑    ↙ ↖
D → C ← E → G ← H

Pop stack + dfs => [D, A, B, C]
Keep popping until E, then dfs => [F, G, E]
Keep popping until H, then dfs: [H]
```

Time complexity: O(V + E)

## Intuition

- In the first step, you do a dfs on a node X then on other non reached Nodes. So the final stack looks like this `[Nodes reachable by X, X, Nodes not reachable by X]`
- In the second step, the graph is reversed, so the stack becomes `[Nodes that can reach X, X, Nodes that can not reach X]`. Therefore, when we arrive at X, all the nodes that can not reach it have already been visited, which guarantees that we found a SCC

## Implementation

### Adjacency matrices

- adj is used for the initial dfs
- rev is the reversed adjacency list, used for the second part of the algo

```python
adj = {}
rev = {}

for s, d in edges:
	if s not in adj:
		adj[s] = []
	if s not in rev:
		rev[s] = []
	if d not in adj:
		adj[d] = []
	if d not in rev:
		rev[d] = []

	adj[s].append(d)
	rev[d].append(s)
```

### Post order dfs

- post order dfs: children are added first

```python
def dfs(src, adj, visit):
    if src in visit:
        return []
    visit.add(src)
    final = []
    for dest in adj[src]:
        final.extend(dfs(dest, adj, visit))
    final.append(src)
    return final
```

### Main algorithm

```python
def kosaraju(adj, rev):
    stack = []
    visit = set()
    for parent in adj.keys():
        if parent in visit:
            continue
        stack.extend(dfs(parent, adj, visit))

    visit = set()
    combs = []
    while stack:
        child = stack.pop()
        if child in visit:
            continue
        combs.append(dfs(child, rev, visit))
    return combs
```
