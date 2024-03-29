---
sr-due: 2024-10-02
sr-interval: 272
sr-ease: 250
reviewed: 2023-07-18
---

#sd

https://grpc.io/

- Built on top of HTTP / 2
- Can not be used natively in the browser: you need a proxy gRPC-web because it needs fine grain control over HTTP/2 that the browser usually does not provide
- typically used for server to server communication
- Faster than REST: instead of sending JSON, it sends Protocol Buffers (object with a schema where fields are ordered) that are serializable in Binary format
- Because it is built on HTTP / 2, allows streaming (one and bidirectional), without having to rely on another protocol such as Websockets
- Less tooling than REST
- does not use status code but error message => custom error handling needed

## Design

- In REST, we access resources (/users, /docs) and the HTTP Method determines the action (GET, POST)
- In gRPC, each endpoint has an action associated (ex: sayHello)
