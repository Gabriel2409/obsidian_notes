---
sr-due: 2023-07-22
sr-interval: 30
sr-ease: 250
---

#sd

## Definition

A Content delivery network is a globally distributed network of proxy servers serving **static content** (images, js code,...) from locations closer to the end user.
The site DNS resolution will tell clients which CDN to contact

- CDN can increase [[Reliability]] and availability: if a server is down, client can ask another server in the CDN network.
- CDN can decrease latency because static content is served from closer locations

## Push CDNs

- Each time server is updated, changes have to be pushed to the CDN network
- Content is uploaded only when it is new or changed, minimizing traffic, but maximizing storage.

Sites with a small amount of traffic or sites with content that isn't often updated work well with push CDNs. Content is placed on the CDNs once, instead of being re-pulled at regular intervals.

## Pull CDNs

- When a user requests content, CDN first tries to serve it. If it is not found (similar behavior to [[Cache]] MISS), it retrieves it from the server and stores it before serving it to the client. It is then available for other clients. That way, different machines in the CDN don't necessarily store the same content
- The Cache-Control header tells the CDN if it can cache the content (if it is set to public)
- A time-to-live (TTL) determines how long content is cached. Pull CDNs minimize storage space on the CDN, but can create redundant traffic if files expire and are pulled before they have actually changed.

Sites with heavy traffic work well with pull CDNs, as traffic is spread out more evenly with only recently-requested content remaining on the CDN.
