---
sr-due: 2023-05-07
sr-interval: 8
sr-ease: 250
---

#dsa #graph

## Definition

Data structure made up of connected nodes

- Vertices = nodes
- Edges = pointers to other nodes
- Directed graph = edges have a direction
- Undirected graph = connections are bidirectional
- weighted graph: edges have an associated weight
- E <= V^2

## Typical representations

### Adjacency list

```python
adj = [[(d01,w01),(d02,w02)],[...]]
```

Each element of the list corresponds to a node. It contains a list of destination/weight.

It can also be represented as a hashmap where key is node and value is list of dest/weights

- Follow up: [[Adjacency list traversal]]

### List of edges

```
edges = [[src, dest, weight],[...]]
```

- compact way to represent the graph
- It is easy to build an adjacency list from the edges

```python
adj = {}
for s,d,w in edges:
	if s not in adj:
		adj[s] = []

	if d not in adj: # should be used both for directed and undirected
		adj[d] = []

    adj[s].append((d,w))
    adj[d].append((s,w)) # undirected graph only
```

Note: the reason we want to add d to the list of keys even in directed graphs is for cases when
we iterate over the adj keys. That way all the nodes are in the dict.

### Matrix

Some graphs can be represented as grids where adjascent elements are connected.
For ex, 0 could represent nodes and 1 absence of connections. Two adjacent 0 would be connected

```python
grid = [[0,0,1,0],
		[0,1,0,0]
	    [0,1,0,0]
	    [0,0,0,0]]
```

- Follow up: [[Matrix traversal]]

## Adjacency matrix

Here elements in the matrix represent the vertex directly.
For ex `adjMarix[i][j] == 1` means there is a connection between vertex i and j.
Contrary to simple matrix, every pair of nodes can be connected.

- weight can be represented as well (value in the matrix)
- If graph is undirected, `adjMatrix[i][j] == adjMatrix[j][i]`

```python
adjMatrix = [[0, 0, 0],
             [1, 1, 0],
             [0, 0, 0]]
```
