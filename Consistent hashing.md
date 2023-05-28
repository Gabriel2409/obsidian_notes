#dsa

## Problem

Usually, when using [[Hashing]] to map a request to a server, we have the following situation:

1. Key is hashed to get an int
2. result is then passed through the % operator to get the correct server

- For ex, in [[Kafka]], when passing a key with a message it is mapped to a partition based on the value of the key
- Other ex: if a [[Load balancer]] maps request to a server based on the IP address (for [[CDN]] for ex)

In all these cases, if the nb of servers change, a key previously mapped to a server can be mapped to a different server even if said server is still online.
This can cause issues if the servers are not stateless (for ex if they all maintain a different redis [[Cache]])

## Definition

Consistent hashing minimizes the traffic change when the nb of servers change.
The idea is to map the key to a position in a circle and also map each server to a position in the circle.
When a key is hashed, you move clockwise until you encounter a server.

To evenly distribute load in case of à server failure, you can put virtual nodes on the cercle (each mapped to an existing server)

It is also possible to implement a flush mechanism to redirect existing keys to the correct server when the number of servers change
