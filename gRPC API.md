---
sr-due: 2024-10-02
sr-interval: 272
sr-ease: 250
reviewed: 2023-07-18
---

#sd

## RPC

RPC allows the execution of a procedure or command in a remote machine. In other words, you can write code that executes on another computer internally in the same way you write code that runs on the current machine. In this approach, the API is more thought of as an action or command. And it is easier to add these functions to extend the functionality.

RPC - /placeAnOrder (OrderDetails order)  
REST - POST /order/orderNumber={} |Order body|

**Strengths:** It is more space efficient than REST, and it makes development easier since the code you write that requires communication to other computers does not require much special syntax.

**Weaknesses:** It can be only used for internal communication. There are complications that can occur, such as timing issues, when you are communicating between machines, and RPC could make this distinction less clear, leading developers to miss corner cases that cause faults in the system.


## gRPC

Implementation of the RPC protocol by google


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
