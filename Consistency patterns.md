---
sr-due: 2023-07-05
sr-interval: 1
sr-ease: 229
---

#sd

## Definition

Note: Consistency here corresponds to the C in [[CAP Theorem]] and has nothing to do with the C of ACID (even though it is unfortunately the same denomination)

When we need to synchronize mutliple copies of the same data, we need to choose how the synchronization process takes place. Depending on this process, we don't provide the same consistency guarantees

### Weak consistency

After a write, reads may or may not see it. A best effort approach is taken. This is typically what happens if you write data to a node and read it from another node : it may or may not have processed the write. There is no guarantee that the other node will process the write.

This approach can be seen in systems such as memcached

### Eventual consistency

After a write, reads will eventually see it. Note that there is no maximum time guarantee. It typically happens within milliseconds but it can take a very long time in some cases.
This is typically the approach that is taken in highly available systems using asynchronous [[Replication]].

### Strong consistency

After a write, reads will see it. That means that a write is acknowledged only if all the replicas are successfully written to. Strong consistency works well in systems that need transaction but it makes the system less available.


### Combining approaches

- It is possible to adopt a mixed approach where you keep some In sync replicas for better consistency while writing to the others asynchronously. For ex [[Kafka]] can do that with the correct settings.