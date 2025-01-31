---
sr-due: 2024-12-05
sr-interval: 316
sr-ease: 250
reviewed: 2023-07-26
---

#sd

## Definition



- REST states for REpresentation State Transfer.
- REST is built on top of HTTP but is not a protocol: it is a set of rules
- REST APIs are stateless: state is not stored on the server but it is stored on the client machine and relevant information is sent as part of the request. This makes horizontal [[Scalability]] easier.
- Most common format is JSON



REST APIs must be modeled based on the resources in the system. A single URI to access that resource is used, and various actions based on the HTTP verbs (GET/PUT/POST/PATCH/DELETE) can be performed on the resources wherever possible.

Strengths: 
- Most used
- structured ways of getting and modifying information from your database.
- Human readable exchange format

Weaknesses: 
- need to write the requests for each type of entity in your database
- Not space efficient (json format)

## Design

- When designing a public facing API, we must be careful to be backwards compatible when possible. For ex, when introducing a new parameter, we can make it Optional. If it is not possible, that's when versionning comes in.
- Example endpoint: `https://api.twitter.com/v1.0/tweet - POST`
- handle pagination with two parameters : limit and offset

