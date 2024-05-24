---
sr-due: 2026-03-31
sr-interval: 678
sr-ease: 250
reviewed: 2023-07-18
---

#sd

Comparing [[B Tree Index|B Trees]] and [[SSTable and LSM Tree |LSM Trees]]

LSM Trees are faster for writes but slower for reads as they have to check different data structures and SS tables at different stages of compaction

## Advantages of LSM Trees

- Write amplification
  - B tree writes every page of data at least twice: once to the [[Write Ahead Log]] and once to the tree page itself. There is also overhead of writing the entire page even if a few bytes in that page changed
  - LSM Trees also rewrite data multiple times due to the repeated compaction and merging of SSTables. However, they are able to sustain higher throughput because the write amplication is lower
- LSM Trees can be better compressed (smaller files on disk)

## Downsides of LSM Trees

- Compaction process can sometimes interfere with read and write at high write throughput
- B tree can offer better transaction semantics: Isolation is implemented using locks on ranges of keys which can not be done in LSM trees as keys can exist in multiple places
- B trees are more well known and provide consistently good performance for many workloads. In constrast LSM trees are more recent and therefore less tested.
