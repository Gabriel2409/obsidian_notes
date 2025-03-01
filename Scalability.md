---
sr-due: 2027-10-07
sr-interval: 1235
sr-ease: 250
reviewed: 2023-07-26
---

#sd

## Definition

Even if a system is [[Reliability | Reliable]] today, it does not mean it will be reliable tomorrow. Scalability is the system ability to cope with increased load.

## Load

To describe the load, we use **load parameters**. When discussing scalability, it is always in relation with these parameters. For example

- a system can be scalable in relation to an increase in the number of requests to a web server but not in relation to the number of writes to a database.
- A system designed to handle 100,000 req/s, each of 1kb is very different to a system designed to handle 5 req/min, each 2Gb in size

## Performance

To discuss [[Performance in a data system | Performance]] in a scalable system, we can ask the following questions:

- If we increase a load parameter and keep the system as is (same CPU, network bandwitdth, memory...), how is the performace affected?
- If we increase a load parameter, how much should resources be upgraded to keep performance intact?

## Coping with load

Good architectures involve a mix of:

- scaling up (**vertical scaling**) = moving to a more powerful machine
- scaling out (**horizontal scaling**) = distributing the load on more machines

Some systems are:

- scaled manually: a human analyses the capacity and decides whether or not to increase the resources
- **elastic**: they automatically add computing resources when load increases (more reactive but also more unpredictable)

Stateless vs stateful:

- stateless systems are easier to scale: we just need to add computing resources
- stateful systems are harder to scale: we need to synchronise the state between the machines. This is why for smaller applications, it may make sense to keep a central database on one node. However, this may be not enough but larger applications.


## More on scaling

### Vertical scaling 
- easiest way to scale initially because it does not require you to change your software architecture.
- Benefit of reducing latency since the communication between different parts of your software architecture is done locally
- However, due to physical hardware limitations, upgrading a computer's hardware past a certain level becomes very expensive or even impossible beyond a certain point
- risk of the single system failing to create a single point of failure

### Horizontal scaling 
Relies on building systems that communicate between multiple computers to store and process information

#### Database scaling
When adding database [[Replication|replicas]] reaches its limit, you need to [[Sharding|shard]] a larger database into smaller databases

#### Compute scaling

Divide the problem into pieces and designate each piece as a job in a queue so that multiple computers can work together in parallel.


### Choosing horizontal vs vertical

While horizontal scaling seems to always be preferred, it is important to consider which services and processes would be more efficient within the same computer.
For ex, with Gmail, there are many inexpensive processes per user. Rather than having each process on a different node, a single node contains a set of users based on geographic proximity. For [[Reliability]] purposes, backup nodes follow and duplicate the primary node in case it goes down.

## Follow up


- [[Shared memory architecture]]
- [[Shared disk architecture]]
- [[Shared nothing architecture]]