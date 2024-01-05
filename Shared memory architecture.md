---
sr-due: 2024-09-10
sr-interval: 250
sr-ease: 250
reviewed: 2023-07-08
---

#sd

## Definition

To [[Scalability|scale]] to higher load, the simplest approach is to buy more powerful machines (called **vertical scaling** or **scaling up**).

Many CPUs, RAM chips and disks can be joined together under one Operating System and a fast interconnect allows any CPU to access any part of the memory or disk.

=> All the components are treated as a single machine.

## Limitation

- Non linear cost (doubling RAM more than doubles cost for ex)
- Non linear performance increase due to bottlenecks (a machine twice as big can not handle twice the load)
- Limited fault tolerance even if high end machines have hot swappable components as we are limited to only one region
