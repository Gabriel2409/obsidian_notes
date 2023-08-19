---
reviewed: 2023-07-14
---

#sd #todo

## Definition

Replication means keeping a copy of the same data on multiple machines that are connected via a network:

- To keep data geographically close to your users (and thus reduce latency)
- To allow the system to continue working even if some of its parts have failed (and thus increase availability)
- To scale out the number of machines that can serve read queries (and thus increase read throughput)

Each node that stores a copy of the database is called a **replica**. We need to ensure that all the data ends up on all the replicas.

3 main types of replication:

- [[Leader Based replication|Single leader]]
- [[Multi leader replication]]
- [[Leaderless replication]]
