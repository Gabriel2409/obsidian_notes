---
sr-due: 2023-06-25
sr-interval: 3
sr-ease: 250
---

#sd

Contrary to [[Shared memory architecture]], Shared disk architecture uses several machines with independant CPUs and RAM. However they still store the data on an array of disks that is shared between the machines.

This can be used for some data warehousing workloads but the overhead of locking still limits the [[Scalability]]

Pros:

- Relatively simple to manage
- Single source of truth
- Cons:
  - Single point of failure
  - Bandwith and network latency
  - Limited scalability
  - Locks on write