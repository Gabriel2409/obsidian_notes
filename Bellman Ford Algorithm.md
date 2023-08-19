---
reviewed: 2023-07-18
---

#dsa #graph #todo

## Definition

Similarly to [[Dijkstra Algorithm]], this algorithm allows to find the shortest path from one node to all of the others. However it also works with graphs that have negative weights on certain edges.

The algorithm initially overestimates the path length from the starting vertex to the other vertices and iteratively relaxes these estimates by funding new paths.
