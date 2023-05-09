---
sr-due: 2023-05-18
sr-interval: 10
sr-ease: 206
---

#dsa #graph

## Definition

A strongly connected component is a portion of a **directed** [[Graph]] in which there is a path from each vertex to another vertex.

```text
A → B     F
↑   ↓    ↗ ↘
D ← C → E ← G → H

[A,B,C,D] [E,F,G] [H]
```

There components can be found with [[Kosaraju algorithm]] or [[Tarjan algorithm]]
