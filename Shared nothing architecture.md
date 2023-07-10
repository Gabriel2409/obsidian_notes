---
sr-due: 2023-07-16
sr-interval: 15
sr-ease: 250
reviewed: 2023-07-08
---

#sd

Contrary to [[Shared memory architecture]] and [[Shared disk architecture]], in shared nothing architecture, each machine or VM (called a **Node**) runs the database software.

This allows for horizontal [[Scalability|scaling]] as each node uses its CPUs, RAM, and disks independently Any coordination between nodes is done at the software level, using a conventional network.

Most data distributed systems use this kind of architecture

Pros:

- Co locating compute and storage avoids networking latency issues
- Generally cheaper to build and maintain
- Improved scaling over shared disk architecture (horizontal scaling)

Cons:

- Storage and compute tightly coupled
- Tendency to overprovision
