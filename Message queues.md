---
sr-due: 2024-09-07
sr-interval: 3
sr-ease: 250
---

#sd 

An asynchronous system is one where the client sends the request and does not wait for the message to be delivered. The client’s message is not immediately sent over to the processing system but instead sent to an intermediary. This intermediary is the message queue or the broker.

- Producers are the systems that produce the messages.
- Consumers are the systems that process those messages.
- Message Queue or topic is the link between the producers and the consumers.
- Producers publish the messages to a named queue, and the queue delivers these messages to the consumer of the queue.

Advantages:
- Prevent overload of the CPU 
- Expensive messages can be held and processing can occur after
- Queues can deliver messages to multiple systems
- Decoupling of client and server by eliminating the need to know the server address

Example: [[Kafka]]