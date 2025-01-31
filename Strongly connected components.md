---
sr-due: 2026-01-04
sr-interval: 502
sr-ease: 206
reviewed: 2023-07-18
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

These components can be found with [[Kosaraju algorithm]] or [[Tarjan algorithm]]
