---
sr-due: 2023-06-21
sr-interval: 14
sr-ease: 250
---

#sd

## Definition

A cache stores result of an expensive operation so that accessing it later can be done faster

## At a single machine level

Data stored on CPU cache can be accessed in nano seconds (vs microseconds for RAM and milliseconds for disk)

## In a distributed system

### Client caching

- Result of an HTTP request can be cached (stored on disk)
- on subsequent request, if it is in cache and not expired (cache HIT), return response from cache without having to to go through the network
- if it is not in cache or expired (cache MISS), perform request on server and, if possible, store result in cache.

Note: GET request (see [[REST API]]) are easier to cache as they are supposed to be idempotent. On the other hand, POST request can modify resources on the server and are therefore a less ideal candidate for caching

### Server caching

Let's consider a basic server with an in memory key value datastore serving as cache (such as redis) and a database stored on disk. The server can cache database reads:

- write-around cache: on write, skip cache. On read, if not in cache or stale (cache MISS), read from disk and store in cache.
- write-through cache: on write, first write to cache then to disk (more work on write but less on read)
- write-back cache: on write, only write to cache + have a background process periodically pushing latest cache entries to disk: faster but less reliable (data loss in case of crash)

### CDN caching

See [[CDN]]

## Eviction policy

- FIFO: elements cached first are deleted first
- [[LRU Cache]] (least recently used) cache: elements accessed the longest time ago are deleted first
- LFU cache (least frequently used cache): elements accessed the least are removed first
