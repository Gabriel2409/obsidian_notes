---
sr-due: 2024-10-01
sr-interval: 271
sr-ease: 250
reviewed: 2023-07-18
---

#sd

## Definition

Type of reverse [[Proxy]] that distribute client requests to servers / databases and returns response to client.

Load balancers make systems more [[Reliability |reliable]] by :

- Preventing requests from going to unhealthy servers
- Preventing overloading resources
- Helping eliminate single points of failure

Load balancers themselves can be horizontally scaled to avoid being a single point of failure.

Note: they can usually handle very high throughput

## Strategies

- Round robin: 1st request goes to 1st server, 2nd to 2nd,...
- Weighted round robin: Each server handle a percentage of traffic
- least active connections
- user location based
- Hashing (IP, request, uid,...), modulo nb of nodes

Note: With hashing, the addition and subtraction of machines leads to the same request being sent to multiple machines. Sometimes, it is important for clients to connect to the same machine when possible, so we can use custom strategies (for ex in an application gateway). We can also use [[Consistent hashing]]

## Types

### Layer 4 LB

- Transport layer LB
- Forwards network packets to and from the upstream server
- Faster but not flexible (strategies limited to simple ones such as round robin, IP hashing,...)

### Layer 7 LB

- Flexible: can use application data to decide where to forward: can involve contents of the header, message, and cookies
- slower:  terminates network traffic, reads the message, makes a load-balancing decision, then opens a connection to the selected server

An application gateway is a layer 7 LB
