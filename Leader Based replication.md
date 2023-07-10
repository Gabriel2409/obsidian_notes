---
sr-due: 2023-07-05
sr-interval: 1
sr-ease: 226
---

#sd #todo

## Overview

Also known as active/passive or master-slave replication.

- One of the replicas is designated as the leader (= master / primary). When clients write to the the db, they MUST send their requests to the leader who writes to its local storage
- The other replicas are followers (= read replicas/ slaves/ secondaries / hot standby). When the leader writes to its local storage, it also sends the data changes to all of the followers as part of a **replication log** (or **change stream**). Each follower then applies the writes in the same order as what is written on the log
- Then when a client wants to read data, it can query any of the followers

In summary from the client point of views, followers are read only and only the leader accepts writes.

This mode of replication is a built-in feature of many databases and is widely used in systems where the read / write ratio is high.

## Synchronous Vs Asynchronous replication

- Synchronous replication:
  - leader waits for acknowledgement of replica before reporting success to user.
  - Advantage: followers have up to date data (strong [[Consistency patterns|consistency]])
  - Drawback: if follower is unavailable, write can not be processed => it is not a good Idea to have only synchronous nodes as a crash of a single node makes the whole system unavailable
- Asynchronous replication: 
    - leader accepts write as soon as it receives it and does not wait for follower acknowledgement (faster write) - Followers are eventually [[Consistency patterns|consistent]] but there is no guarantee on the time it takes
    - Durability is weakened because we can lose data if leader crashes
- Semi synchronous:
    - Usually one synchronous node and the rest as asynchronous node

## Set up new followers

- Option 1: lock the db in order to copy it => goes against availability goal
- Better option:
  - take a consistent snapshot of the leader's db (usually this can be done without lock)
  - Copy snapshot to follower node
  - Follower then connects to leader and requests changes (snapshot must be associated with exact position in replication log, called **log sequence number**, **binlog coordinates**...)
  - Once follower has processed the backlog, we say it has caught up

## Node outages

### Follower failure: catch up recovery

When follower crashes or network between leader and follower is temporarily interrupted

Recovery is easy: by looking at its replication log, the follower knows the last transaction that was processed before it was disconnected. It can then request all the data changes from the leader, similarly to what happens when setting up new followers

### Leader failure: Failover

Failover is the **process of promoting a follower to leader when the current leader is unavailable**:

- Nodes must then consume data from the new leader
- This can be done manually by an administrator
- This can also be done automatically:
  - Determine that leader has failed: as many things could go wrong (crashes, power outages, network issues), most systems use a timeout
  - Choose a new leader: could be done through an election process or could be appointed by a previously elected controller node. Usually the ideal candidate is the follower with the most up to date data
  - Reconfigure the system to use the new leader: clients must send their write requests to the new leader and if previous leader comes back online,not realizing it is not leader anymore, the system needs to ensure it becomes a follower
- Multiple things can go wrong
  - in asynchronous replication, **most up to date follower might not have all the data**. Even when the old leader comes back online, most systems just discard these unreplicated writes, which can be dangerous if other storage systems need to be coordinated with the db
  - in certain scenarios, we can have a **split brain** (two nodes believe they are leader), which can lead to data loss/corruption. Systems should automatically shut down one of the nodes
  - Choosing the **correct timeout** is difficult. If it is too long, you can have high latency. If it is too short, you can have unnecessary failovers, straining the system even more.

## Types of replication logs

- Statement based replication
  - leader logs every write request. Followers parse it and execute SQL statement
  - any statement that calls a nondeterministic function needs to be replaced
  - If statements use an autoincrementing column or if they depend on existing data, they MUST be executed in the same order
  - Potential problems with non deterministic side effects
- Write ahead log shipping
  - See [[SSTable and LSM Tree]] and [[B Tree Index]], we can use the WAL log (append only sequence of bytes) to build the replica on another node
  - Main disadvantage is that the log describes the data at a very low level (details of which byte changed on which disk blocks), which makes replication coupled to the storage engine
  - This prevents zero-downtime upgrades as leaders and followers must be on the same software version
- Logical (row-based) log replication
  - Contrary to a physical log, a logical log is decoupled from the storage engine
  - When inserting rows, it contains the new values of all columns, when deleting, it contains info of the deleted row, etc...
  - A logical row is easy to parse and can allow replicas to run on different software versions or even different storage engines
- Trigger based replication
  - a trigger lets you register application code, for example to log the changes in another table so that it can be processed by an external program
  - More flexible BUT Higher overhead and more prone to bug

## Potential problems (specifically with asynchronous replication)

As leader based replication requires all writes go to the leader, it is good for workloads that consist mostly of reads.
To create a read-scaling architecture, add more asynchronous followers.
When reading from an asynchronous follower, you can see out of date data (followers are only eventually consistent).

### Read your own rights

- If you write to the leader and immediately read from a follower, you may not see the written data because replication is asynchronous. For ex, if a user updates his profile, he may not see the updated information on his profile page.
- To avoid that, we need read-after-write consistency. In the ex above, we could make every user read his own profile from the leader and profile of others to a follower
- Note that if everything can be edited by the user, we will lose the advantage of read-scaling by forcing all reads to go through the leader
- Another possibility is to make the client remember the timestamp of his most recent write and only read from followers who have caught up to this timestamp (timestamp could be logical, such as a sequence nb)
- There is additional complexity if a user can connect through multiple devices

### Monotonic reads

- If you read from follower 1, then follower 2 and follower 2 is lagging behind follower 1, you may have the impression that some data disappeared
- Monotonic reads is a guarantee against this kind of anomalies. After a read, you will not read from a follower that is lagging behind. This is not strong consistency as the data can still be out of date but, at the very least, data does not disappear between reads
- This can be achieved by making sure a user reads from the same replica (for ex by choosing a replica which depends on the hash of a user ID)

### Consistent Prefix Reads

- If a write depends on a previous write, because replication is asynchronous, you may see them out of order when reading from a replica
- consistent prefix reads is a guarantee that if a sequence of writes happen in a given order, then anyone reading those writes will see them in the same order
- This is very hard to achieve in partitioned (sharded) databases because there is no global ordering (partitions operate independently)
