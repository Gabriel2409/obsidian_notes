---
sr-due: 2024-08-02
sr-interval: 212
sr-ease: 230
reviewed: 2023-07-06
---

#sd

## Definition

- C = Consistency (see [[Consistency patterns]], nothing to do with C in ACID)
- A = Availability
- P = Partition tolerance (partition here has NOTHING TO DO with [[Sharding]])

CAP theorem is often misunderstood as : for a given system, choose 2 of the 3.

In reality, connection problems within the network WILL always happen at some point creating a partition where different zones of the network can not communicate with each other.

So what CAP theorem tells us is that we have to choose between:

- Either an highly available system that can become inconsistent when there is a network partition
- Either a very consistent system that can become unavailable when there is a network partition

## PACELC

CAP theorem is limited because it does not tell us about what to favor in a system when there is no network partition. A newer version is PACELC: Given P, choose A or C. Else Favor Latency or Consistency.

The first part is the CAP theorem correctly rewritten.
The second part tells us that even when there is no problem, favoring consistency will cause an increase in latency (because write must be acknowledged by several [[Replication|replicas]]) and favoring latency is done at the expense of consistency (write only acknowledged by one node)
