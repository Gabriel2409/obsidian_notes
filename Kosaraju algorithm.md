---
sr-due: 2024-12-27
sr-interval: 215
sr-ease: 190
reviewed: 2023-07-18
---

#dsa #graph

## Definition

Kosaraju algorithm is an algorithm to detect [[Strongly connected components]] In a directed [[Graph]]

- Step 1: building the stack
  - Start on a random node and perform a post order dfs to get all the reachable nodes in a stack
  - Repeat operation on all nodes that were not reached to complete the stack
  - The output of this step is a stack containing all the nodes with children before parents
  - note: step 1 follows the same logic as [[Topological sort]]
- Step 2: Finding the SCCs:
  - Pop the stack and perform a dfs on the resulting node but on the **REVERSED** graph to get a strongly connected component. Note that this dfs can be a preorder or a post order dfs, it does not matter
  - Keep popping the stack to get the other components (ignore the nodes that were already visited)

```text
Step 1

A → B     F
↑   ↓    ↗ ↘
D ← C → E ← G → H

Start on a random node, for ex E
dfs => Example stack = [H, G, F, E]
Go to a node not already visited, for ex B
stack = [A, D, C, B]
Final stack = [H, G, F, E, A, D, C, B]
```

```text
Step 2: dfs on reversed graph

A ← B     F
↓   ↑    ↙ ↖
D → C ← E → G ← H

Pop stack + dfs => [C, D, A, B]
Keep popping until E, then dfs => [F, G, E]
Keep popping until H, then dfs: [H]
```

Time complexity: O(V + E)

## Intuition

- In the first step, you do a dfs on a node X then on other non reached Nodes. So the final stack looks like this `[Nodes reachable by X, X, Nodes not reachable by X]`.
- In the second step, the graph is reversed, so the stack becomes `[Nodes that can reach X, X, Nodes that can not reach X]`. Therefore, when we arrive at X, all the nodes that can not reach it have already been visited, which guarantees that we found a SCC. Then when you start the dfs on X, you know that each node reachable by X can also reach X, making it part of the same SCC.

Note: Here, the logic can be applied to nodes that were selected to start the dfs, ie the first node then the node after the first dfs is over, then the next... IT DOES NOT WORK FOR ALL THE NODES (easy to understand with 2 nodes pointing to each other)

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
