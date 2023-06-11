#sd #todo

## Overview

Also known as active/passive or master-slave replication.

- One of the replicas is designated as the leader (= master / primary). When clients write to the the db, they MUST send their requests to the leader who writes to its local storage
- The other replicas are followers (= read replicas/ slaves/ secondaries / hot standby). When the leader writes to its local storage, it also sends the data changes to all of the followers as part of a **replication log** (or **change stream**). Each follower then applies the writes in the same order as what is written on the log
- Then when a client want to read data, it can query any of the followers

In summary from the client point of views, followers are read only and only the leader accepts writes.

This mode of replication is a built-in feature of many databases and is widely used in systems where the read / write ratio is high.

## Synchronous Vs Asynchronous replication
