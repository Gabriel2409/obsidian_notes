---
sr-due: 2023-08-10
sr-interval: 43
sr-ease: 250
---

#sd
## Definition

- REST states for REpresentation State Transfer.
- REST is built on top of HTTP but is not a protocol: it is a set of rules
- REST APIs are stateless: state is not stored on the server but it is stored on the client machine and relevant information is sent as part of the request. This makes horizontal [[Scalability]] easier.
- Most common format is JSON

## Design
- When designing a public facing API, we must be careful to be backwards compatible when possible. For ex, when introducing a new parameter, we can make it Optional. If it is not possible, that's when versionning comes in.
- Example endpoint: `https://api.twitter.com/v1.0/tweet - POST`
- handle pagination with two parameters : limit and offset