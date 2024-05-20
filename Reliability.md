---
sr-due: 2025-09-22
sr-interval: 491
sr-ease: 250
reviewed: 2023-07-26
---

#sd

## Definition

- A system is reliable if it continues to perform correctly even in the face of adversity.
- In particular, systems that anticipate faults and can cope with them are called **resilient** or **fault tolerant**

## Type of faults

### Hardware faults

Hardware can malfunction (crash, black outs...). For ex, Hard disks mean time of failure (MTFF) of about 10 to 50 years => 1 disk / day dies in a cluster of 10,000 disks.

Until recently, redundancy of hardware components was enough (restoration from backups + small downtime was used)

As applications use more and more machines, software fault tolerance techniques that can deal with the loss of entire machines are now used. Advantages:

- No need for planned downtime. If system can tolerate node failures, these nodes can be patched one at a time without the whole system going offline (rolling upgrade)

### Software errors

- Bugs that cause crash with certain types of inputs
- runaway process using up a shared resource (CPU, memory, disk space...)
- A service that becomes unresponsive causing the system to hang
- cascading failures where a small fault in a component triggers a chain reaction

To minimize them:

- Process isolation
- Thorough testing
- Induced frequent crash of individual components to check the system fault tolerance
- Healthchecks

### Human errors

Most of the errors in software are due to a mistake by a human operator. In order to minimize them:

- Design system that minimize opportunities for error
- Allow easy rollbacks
- Have a dedicated sandbox environment that replicates prod environment

## Reliability evolution

- To make sure a system remains reliable, we must ensure it is [[Scalability | Scalable]] if we anticipate a load increase and [[Maintainability| Maintainable]]
