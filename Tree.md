---
sr-due: 2027-01-06
sr-interval: 961
sr-ease: 270
reviewed: 2023-07-11
---

#dsa #tree

In graph theory, a tree is an unidirected [[Graph]] in which any two vertices are connected by exactly one path

In practice, it is a hierarchical data structure consisting of nodes connected by edges. Each node can have multiple children and each of these children can have their own children.

```mermaid
graph TD;
A-->B
A-->C
A-->D
D-->E
D-->F
E-->G
```

In a Tree, `E = V - 1` with `E` the nb of edges and `V` the nb of vertices

