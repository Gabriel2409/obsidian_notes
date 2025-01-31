---
sr-due: 2024-11-16
sr-interval: 298
sr-ease: 250
reviewed: 2023-07-26
---

#sd

## Definition

- Not a protocol: Built on top of HTTP but only uses HTTP POST request because we need to send data in a body
- Single endpoint where you can do a query (read) or mutation (update, delete,...). This solves overfetching and underfetching typical in [[REST API]].
  - In REST, if we need to access resources from multiple endpoints, we must do a request on each of these endpoints and usually fetch a lot of data we don't need.
  - Example: we want to fetch all the comments of a given user. In REST, we fetch all the information of the user to retrieve its id and then use the id to fetch the relevant comments. When fetching the user, we got all the information on the user, even though we only needed the id. Of course in REST we could add custom logic but it requires server side changes.
- As POST request are not idempotent, GraphQL request can not be cached as well as REST
- Endpoints have a schema that we must respect


**Strengths:** GraphQL works particularly well for customer-facing web and mobile applications, because once you set up the system, frontend developers can craft their own requests to get and modify information without requiring backend work to build more routes.

**Weaknesses:** There is initially some upfront development work required to set up this communication system, both on the frontend and backend. It is also less friendly for external users when compared with REST APIs, where documentation can be generated automatically. In addition, GraphQL is not suitable for use cases where certain data needs to be aggregated on the backend.