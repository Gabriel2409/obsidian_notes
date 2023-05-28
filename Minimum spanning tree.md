---
sr-due: 2023-06-05
sr-interval: 16
sr-ease: 230
---

#dsa #graph

## Definition

A minimum spanning tree (MST) is **a subset of the edges of a connected, edge-weighted [[Graph]] that connects all the vertices together without any cycles and with the minimum possible total edge weight**. It is a way of finding the most economical way to connect a set of vertices.

It is a [[Tree]] because each node is connected to another with only one path (E = V - 1)

In the example below, we keep the edges A-C and B-C because they have a lower weight. The final MST has 2 edges for 3 vertices

```text
      A         A
   3 / \ 2 =>    \
    B — C     B — C
      1
```

To find the MST In a weighted undirected [[Graph]], see [[Kruskal algorithm]] algorithm and [[Prim algorithm]]
