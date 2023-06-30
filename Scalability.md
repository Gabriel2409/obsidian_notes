---
sr-due: 2023-09-07
sr-interval: 70
sr-ease: 250
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


- [[Shared memory architecture]]
- [[Shared disk architecture]]
- [[Shared nothing architecture]]