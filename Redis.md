#redis

Redis stands for Remote dictionary server

Easily run redis-stack: `docker run -d --name redis-stack -p 6379:6379 -p 8001:8001 redis/redis-stack:latest` then access redis-insights on port 8001

Interview questions: https://medium.com/p/77d064e0d721

# Getting started

- https://university.redis.com/
- https://app.redislabs.com
- https://redis.io/docs/install/install-redis/install-redis-on-linux/

On linux, start redis-server with `systemctl start redis-server` and connect with `redis-cli`

- get data location: `redis-cli config get dir`: in ubuntu, it returns `var/lib/redis` and data is in `dump.rdb`
- get config location: `redis-cli INFO | grep config_file`: in ubuntu, `/etc/redis/redis.conf`

Alternatively, run `docker run --name redis-container redis` followed by `docker exec -it redis-container redis-cli` OR you can do `docker run -p 6379:6379 redis` followed by `redis-cli` if you have installed redis-tools on your machine

Note: you can create a free account on redislabs and download RedisInsights

# Redis basic datastructures

## Keys

https://redis.io/docs/manual/keyspace/

- unique in the whole db
- binary safe and therefore case sensitive
- up to 512 MB
- in a flat key space (no automatic namespacing)

Note: Redis also comes with logical databases that provide a way to organize data within the redis instance (serves as a sort of namespace). They still use the same underlying hardawre so don't provide real isolation
Logical db are 0 indexed.
Many tools and frameworks assume database 0 is used. We will focus on only db 0 here

- [SET](https://redis.io/commands/set/): Sets a key with a value

```bash
# customer:1000 key now is associated to value Fred, redis returns OK
SET customer:1000 Fred
# Only set the key if it exists already, redis returns either OK or (nil)
SET customer:1000 john xx # use nx for non existence check
# Retrieve previous value when setting the key
SET customer:1000 paul GET # returns prev value, here john
# Set an expiration time on the key with EX|PX|EXAT|PXAT depending
# on whether to use seconds, milliseconds or timestamps (in ms or s)
SET customer:1000 michel PX 500 # key expires in 500 ms
```

- [GET](https://redis.io/commands/get/) Get the value of key. If key does not exist: return nil: `GET customer:1000`
- [DEL](https://redis.io/commands/del/): Deletes a key (or a list of key): blocking operation

```bash
DEL customer:1000 #returns nb of keys deleted, here 1
```

- [UNLINK]: similar to DEL but performs the actual memory reclaiming in a different thread, so it is not blocking, while DEL is.
- [KEYS](https://redis.io/commands/keys/): `KEYS <glob_pattern>`: finds all the keys matching the pattern and return their values. Blocking, do not use in production
- [SCAN](https://redis.io/commands/scan): similar to keys but safe for production. O(1) for every call and each call returns a slot id

```bash
# iterates on all keys using a cursor
# returns matches and next slot id to call to continue scanning
# you may need many calls to get all matches but it is safe
# when scan returns a slot_id of 0 you have no more key to iterate over
# NOTE: first scan should be made with 0
scan <slot_id> MATCH <pattern> COUNT <nb_keys_per_call>
```

- [EXISTS](https://redis.io/commands/exists/): check existence of key, returns nb of successes: `EXISTS customer:1000`

- Set the expiration time of the keys after creation with [EXPIRE](https://redis.io/commands/expire/), [EXPIREAT](https://redis.io/commands/expireat/), [PEXPIRE](https://redis.io/commands/pexpire/), [PEXPIREAT](https://redis.io/commands/pexpireat/)
- Inspect time to live of the key with [TTL](https://redis.io/commands/ttl/) and [PTTL](https://redis.io/commands/pttl/)
- Remove expiration with [PERSIST](https://redis.io/commands/persist/)

- [TYPE](https://redis.io/commands/type/)Get the type of the key
- [OBJECT](https://redis.io/commands/object/): depends on subcommand: `OBJECT help`

### Note on redis cluster sharding

Redis Cluster does not use [[Consistent hashing]], but a different form of sharding where every key is conceptually part of what we call a **hash slot**.

There are 16384 hash slots in Redis Cluster, and to compute the hash slot for a given key, we simply take the CRC16 of the key modulo 16384.

Every node in a Redis Cluster is responsible for a subset of the hash slots, so, for example, you may have a cluster with 3 nodes, where:

- Node A contains hash slots from 0 to 5500.
- Node B contains hash slots from 5501 to 11000.
- Node C contains hash slots from 11001 to 16383.

## Strings

https://redis.io/commands/?group=string

- binary safe sequences of bytes (no assumption of content): `SET user:101:time-zone UTC-8`: here UTC-8 is a redis string
- caching, for ex, you can cache a serialized JSON object with an expiry time
- implementing counters with support for increment/decrement with [INCR](https://redis.io/commands/incr/) and [INCRBY](https://redis.io/commands/incrby/): note that it returns an error if value is not an integer. However, if the key does not exist, it will assume value was 0 and do the increment

### About Bitmaps

https://redis.io/commands/?group=bitmap

- Not a datatype but a set of operations you can perform on strings. Indeed strings in redis are binary safe values
- can be used to store data as diverse as histograms of hourly counters or access bits on a file or seat reservation...
- Commands on bits start from left to right, so an offset of 0 will target the leftmost bit

```bash
SETBIT bitkey 0 1 # sets 1 at offset 0
GETBIT bitkey 0 # returns 1
GET bitkey 0 # returns "\x80"
```

- `BITCOUNT` counts the nb of bits in a string
- `BITPOS` finds the first 0 or 1 BIT in a string
- `BITOP` performs operation on bit

```bash
# performs operations on string by treating them as bit arrays
BITFIELD mykey GET u8 #1 -- Here #1 is the offset in multiple of encoding as it starts with a #
```

## Hashes

https://redis.io/commands/?group=hash

- Implemented as a [[Hashmap]]
- collection of mutable field value pairs like a mini key value store within a key
- schemaless
- can only contain strings only (not recursive, no concept of nested hierarchy)
- Typical usecase: rate limiting, session store
- most are O(1) except HGETALL which is O(n)
- set key value pair with [HSET](https://redis.io/commands/hset/) , delete with [HDEL](https://redis.io/commands/hdel/) , get field with [HGET](https://redis.io/commands/hget/) , and all fields with [HGETALL](https://redis.io/commands/hgetall/), increment a field with [HINCRBY](https://redis.io/commands/hincrby/) ,...

```bash
HSET user:1 name jo color blue
HSET user:1 name john age 18 # updated name ans set age
HDEL user:1 color
HGET user:1 name # john
HINCRBY user:1 age 1
HGETALL user:1
```

## Lists

https://redis.io/commands/?group=list

- Implemented as a Doubly [[Linked list]]
- ordered collections of strings, not nested
- duplicates are allowed
- typical usecase: activity stream, inter process communication
- [LPUSH](https://redis.io/commands/lpush/) and [RPUSH](https://redis.io/commands/rpush/) to add elements
- [LPOP](https://redis.io/commands/lpop/) and [LPOP](https://redis.io/commands/lpop/) to remove elements
- [LLEN](https://redis.io/commands/llen/) to get the length
- [LINDEX](https://redis.io/commands/lindex/) to access element: `LINDEX mykey 0` accesses first element
- [LRANGE](https://redis.io/commands/lrange/) : `LRANGE mykey 0 -1` retrieves all elements: contrary to python, we include last element
- [LTRIM](https://redis.io/commands/ltrim/) for lists to KEEP only certain elements

## Sets

https://redis.io/commands/?group=set

- Implemented as a [[Hashmap|Hash table like structure]]
- unordered collection of strings without duplicates
- no nested hierarchy
- support intersection, difference, union
- usecases: tag cloud, unique visitors,
- [SADD](https://redis.io/commands/sadd) to add element
- [SCARD](https://redis.io/commands/scard) to get nb of elements
- [SISMEMBER](https://redis.io/commands/sismember) to check if element is in set
- [SMEMBERS](https://redis.io/commands/smembers) to get all members but as usual, prefer `SSCAN`
- `SINTER set1 set2` to get intersection
- remove element with `SREM` or `SPOP` (SPOP removes a random element)
- Sets can be set to expire
  - Example: we maintain two sets (current period and next period). Each time an event occurs it is added to the two sets. That way, when first set expires, next period set contains elements that were added in previous period

## Sorted sets

https://redis.io/commands/?group=sorted-set

- Implemented as a combination of a [[Hashmap|Hash table]] and [[Skip list]]
- usecase: priority queue, low latency leaderboard, secondary indexing
- Note: more like ordered key-value store than sets: you store each element with a score
- `ZADD myset myscore mymember`: here score must be int or float
- `ZINCRBY` to increment score
- `ZRANGE` to access top / bottom elements (with `REV`)
- `ZRANK` to get rank
- `ZCOUNT` to count nb of elements between two scores
- `ZREMRANGEBYRANK` to REMOVE certain elements, very similar to `LTRIM` for lists but focuses on elements to remove instead of elements to keep
- `ZINTERSTORE` to store result of intersections into a destination set. Possibility to add aggregation and weights. NOTE: you can use this with a standard set as well, in this case, you can specify the score of all the elements in the set with the WEIGHT parameter

## Geospatial

https://redis.io/commands/?group=geo
https://en.wikipedia.org/wiki/Geohash
https://en.wikipedia.org/wiki/Haversine_formula

- For each longitude and latitude pair, a GeoHash is computed = 52 bit integer value which encodes positions
- All longitudes from -180 to 180
- LIMITATION on latitude from -85.05 to 85.05
- Geo spatial data is actually stored as a sorted set where the score is the geohash so you can use all sorted sets commands

```bash
GEOADD geokey 10 20 mystation #longitude is first, latitude is second
GEOHASH geokey mystation # returns the STANDARD geohash, not the 52 bit integer redis representation
GEOPOS geokey mystation # returns 10 20
```

- `GEODIST` gives distance between two members. Note: redis is more precise towards the equator than the poles as it assumes the earth is a perfect sphere and use the Haversine formula
- `GEOSEARCH` provides multiple ways to search within an area

**DANGER when performing set operations**: ZUNIONSTORE and ZINTERSTORE, would sum up the scores which would mean you change the position of elements, be sure to use the MIN or MAX aggregation operator: `ZINTERSTORE dest 2 geo1 geo2 AGGREGATE MIN`. Note: if you need distance between two elements in different sets, you can first join the sets in a new set and then perform the distance

# Redis: extra info

## Search on multiple criteria

Traditional approach:

- Secondary indexes
- Full text searches

In redis, we don't have secondary indexes.
Method 1: encode all object as string, then match all potential objects, deserialize value and inspect it to see if it matches: linear complexity

Method 2: we can use Faceted search (Inverted index), which allow to search and filter using multiple criteria. Idea is to create multiple sets where key is attribute and values are all elements matching said attribute. Then we can just use SINTER to intersect multiple sets and get all elements matching the corresponding attributes.
Problem is if attribute can have a large nb of values, we must create a lot of sets

Method 3: Redis does not allow us to create compound indexes. However, we can mimic this with [[Hashing]]. For ex, we create a hash for criteria1, then another for criteria1 and criteria2. We then use the previous method wehre we create a set with only the matching elements.

## Executing lua scripts

```bash
EVAL "return KEYS[2]" 3 key1 key2 key3 # returns key2
SCRIPT LOAD "return KEYS[2]" # returns SHA1
EVALSHA <sha1> 3 key1 key2 key3 # returns key2
```

When the execution time is exceed (by default 5 seconds), then the Redis server will start to accept new commands.
However, it will only accept a limited set of Administration commands. Any other command, for example a GET, will have a BUSY response returned.

# Interact with data

## Transactions

https://redis.io/commands/?group=transactions

Redis transactions are different from [[Transaction]] in a standard db. Note that despite being a NoSQL db, redis provides [[ACID]] guarantees

- Redis uses a single threaded event loop for command execution
- Commands are queued
- Redis guarantees each transaction is atomic
- Nested transactions are not supported
- As long as transaction is not complete, others don't see the updated value (isolated changes)

```bash
MULTI # starts the transaction
SET event:Judo 100 # returns QUEUED
INCR event:Judo # QUEUED
GET event:Judo # QUEUED
EXEC # executes all queued operations
```

Here, if another connection had tried to read event:Judo before the execution of the transaction, it would see the initial value.
Note: to abort the transaction and discard the pending commands, use `DISCARD`

Note: there is no dirty read in redis because redis ensures that all commands in a transaction are executed with no other client commands in between. There is no commit / rollback like in traditional dbs.
In fact there are two kind of errors in a transaction: a command may fail to be queued (ex syntax error) or a command may fail after EXEC (for ex trying to pop a string)

In fact, redis will catch errors before EXEC but, when calling EXEC, all commands are executed even if they return an error

In redis, there is no concurrent writes either as commands are queued. Redis can handle multiple concurrent clients but will still execute commands sequentially

To make the EXEC conditionl use `WATCH` (optimistic locking): On EXEC, redis will perform the transaction ONLY if non of the WATCHED keys were modified
Ex implement `ZPOP`:

```bash
WATCH zset
element = ZRANGE zset 0 0
MULTI
ZREM zset element
EXEC
```

After EXEC, all keys are unwatched but you can also call `UNWATCH` directly

## Pub/Sub

https://redis.io/commands/?group=pubsub

- Redis' Pub/Sub exhibits **at-most-once** message delivery semantics: messages are only sent once by the server and if a client fails to handle it, it is lost. If we need stronger guarantees => Redis streams
- message must be a redis string
- Pub/Sub has no relation to the key space. It was made to not interfere with it on any level, including database numbers. Publishing on db 10, will be heard by a subscriber on db 1.

```bash
# symple syndication
PUBLISH channel message # publishes a message on the channel
SUBSCRIBE channel # subscribes to the channel
UNSUBSCRIBE channel # unsubscribes
# pattern syndication
PSUBSCRIBE channel-* # subscribes to all channels matching the pattern
PUNSUBSCRIBE pattern # same with unsubscribe
# admin
PUBSUB NUMSUB channel # returns nb of subscribers on the channel
PUBSUB NUMPAT # returns nb of unique pattenrs subscribed by clients with PSUBSCRIBE
```

- For a single redis node, ordering of messages in a channel is guaranted
- Pattern matching is done only at the subscription moment, if you add new channels that match the pattern afterwards, they are not subscribed

- It is also possible to use shard channels: `SSUBSCRIBE`, `SPUBLISH`, `SUNSUBSCRIBE`: sharding algorithm used is the same as algorithm used to shard keys

# Redis Streams

https://redis.io/commands/?group=stream

- Acting on data in real time
- redis datastructure, You can use all redis commands such as `EXPIRE`, `DEL`, ...
- behaves like an append only log but redis allows to delete entries
- Each entry is structured as a set of key value pairs, just like a redis hash
- Entries have unique Ids, by default timestamp prefix and sequence nb suffix so that stream entries are effectively indexed by time
- Streams support id based range queries
- Streams can be consumed and processed by multiple distinct sets of customers (consumer groups)
- Stream supports both blocking and non blocking mode
- Stream are immutable in the sense that their entries can not be modified. BUT they can be deleted. In fact redis won't remove it physically but mark it as removed. Moreover, you can trim streams
- Redis does not compress the actual data but may compress the field names, for ex when a field name appears several times, instead of storing it directly, redis internally uses pointer like logic
- Contrary to PubSub that don't store any data, streams are data structures and store data in memory

### Producer and consumer

Adding an entry to a stream is O(1). Accessing any single entry is O(n), where _n_ is the length of the ID. Since stream IDs are typically short and of a fixed length, this effectively reduces to a constant time lookup. For details on why, note that streams are implemented as [radix trees](https://en.wikipedia.org/wiki/Radix_tree)

- `XADD` is the only redis command that adds data to a stream: use `*` to automatically specify the id, for ex `XADD mystream * field1 value1 field2 value2`. By default id is `<millisecondsTime>-<sequenceNumber>`. If you specify id, it must be greater than last stream id. Note: redis avoids travelling back in time by setting value at the greatest timestamp if its internal clock is lower.
- `XREAD` reads one or more entries, starting at a given position and moving forward in time. Note that it does not allow field level filtering, it reads the whole message
- `XRANGE` (and `XREVRANGE`) returns a range of entries between two supplied entry IDs. Note there is no `XGET`, to get a single entry we use XRANGE with the specified id twice
- Incomplete ids: right part is considered 0, for ex id 458782 is equivalent to 458782-0
- Special ids for XRANGE: `-` and `+` mean respectively the minimum ID possible and the maximum ID possible inside a stream
- Contrary to XRANGE, XREAD returns messages that come STRICTLY AFTER the specified id

```bash
# return two elements per streams
XREAD COUNT 2 STREAMS stream1 stream2 0-0 0-0
# returns all elements
XRANGE somestream - +
# iterating on the stream
XRANGE writers - + COUNT 2 # returns first 2 elements
XRANGE writers (1526985685298-0 + COUNT 2 # returns 2 elements starting from (1526985685298-0 (excluded)
```

NOTE for `XREAD`: If the **BLOCK** option is not used with `XREAD`, the command is synchronous, and can be considered somewhat related to [`XRANGE`](https://redis.io/commands/xrange) but

- This command can be called with multiple streams if we want to read at the same time from a number of keys.
- more suited in order to consume the stream starting from the first entry which is greater than any other entry we saw so far
  In order to avoid polling at a fixed or adaptive interval the command is able to block if it could not return any data, according to the specified streams and IDs, and automatically unblock once one of the requested keys accept data.

```bash
# wait for 1000 ms to receive data and returns nil if no new data appeared
XREAD BLOCK 1000 STREAMS mystream 1526999626221-0

# use the $ id if you are only interested in entries that are added after you explicitely blocked
# Note that blocking stops as soon as you have a message even if you specified a larger count
XREAD BLOCK 5000 COUNT 100 STREAMS mystream $
# after that you must use XREAD with the id you obtained and not the $ sign because
# other messages may have appeared in the meantime
```

- `XLEN` gives the length of the string: `XLEN mystream`
- `XDEL` deletes messages on stream. Note that even if the stream length is 0, it can still exists, contrary to other data structures that are deleted
- It is possible to trim streams to reclaim memory (synchronous operation with linear time complexity)

```bash
XTRIM mystream MAXLEN 5 # trims all but latest 5 messages
XTRIM mystream MAXLEN ~ 1000 # almost exact trimming
# or directly when adding a message, you can trim after
XADD mystrem MAXLEN ~ 5 * field value
```

### Consumer groups

- If rate at which messages arrive is greater that the speed at which they can be processed (consumer is too slow), we can group consumers in a consumer group. Within a consumer group, each consumer will only process a portion of the messages. More formally, a consumer group is an implementation of the logical consumer for a single stream which can be made up of any number of physical consumers

```bash
# set consumer group to track existing mystream starting from now
XGROUP CREATE mystream mygroup $
# set consumer group to track new stream starting from id 0
XGROUP CREATE newstream group2 0 MKSTREAM
# find all groups watching a stream
# you notably get the nb of pending messages (delivered but not acknowledged)
# the last-delivered-id, the total nb of entries read by the group
# and the lag (nb of messages in the stream not yet delivered)
XINFO GROUPS mystream
```

- Add consumer to a group

NOTE: use `CLIENT SETNAME myconnection` to set the name of a connection (useful when having several clients). A good practice is to give the consumer group the same name as the client. That way the command `CLIENT LIST` will be helpful
To read from a consumer group, we don't use `XREAD` but [XREADGROUP](https://redis.io/commands/xreadgroup/)

```bash
# requires both group and consumer name
# consumer will be created if does not exist
# COUNT and BLOCK and STREAMS are similar to XREAD
# special id > corresponds to next undelivered message
XREADGROUP GROUP mygroup myconsumer COUNT 1 BLOCK 1000 STREAMS numbers >
```

Redis keeps track of which messages have been delivered to which consumer group members using a Pending Entries LIST (PEL) at the consumer level. Under normal conditions, removal from the PEL is done once the customer acknowledges successfully processing the message

```bash
# provides infos on customer and where they are in the stream
# you notably get the pending key = number of entries in the PEL
XINFO CONSUMERS gmystream mygroup

# get all pending messages in a consumer from the PEL
# note that even if there are new messages in the stream,
# it only shows pending messages
# Only way to add new messages to the PEL is with id >
XREADGROUP GROUP mygroup myconsumer STREAMS mystream 0
```

- When a consumer succeeds processing, it acknowledges it with `XACK`. It is then removed from the PEL, which means it won't appear with the above command anymore

```bash
# acknowledge 1 or more ids
# note that we don't require the consumer name, which means acknowledgement
# can be done by another process
XACK mystream mygroup <id>
```

Note: you can also use consumer groups with at most once semantics (No PEL). TO do so use `NOACK` with XREADGROUP

- Each group has a specific position in the stream (last-entry-id). It is possible to manually change it with SETID

```bash
# id=0 -> reset group | id=$ -> place group at the end of stream
XGROUP SETID mystream mygroup <id>
```

- There is no automatic cleanup. Deletion should be manual

```bash
# deletes group and customers
XGROUP DESTROY mystream mygroup
# deletes a consumer, returns nb of pending msgs
XGROUP DELCONSUMER mystream mygroup myconsumer
```

- If we want to reassign pending messages prior to deletion, we need to use `XPENDING` and `XCLAIM` first. If you delete a message that had pending messages, and if you run `XINFO GROUPS mygroup`, you will see that the pending field decreases, which means messages were not delivered and redis is not waiting for acknowledgement anymore.

```bash
# returns nb of pending messages per consumer
XPENDING mystream mygroup
# returns id of all pending messages + info on consumer
# last fields are time from delivery and nb of times delivered
XPENDING mystream mygroup - + 3
# note that running the below line resets the time
# since delivery and increases the nb of times counter
XREADGROUP GROUP mygroup myconsumer COUNT 1 STREAMS mystream 0

# claim will only succeed if elapsed time since delivery
# is greater than min-idle-time
# below line claims the message of id 1705335383125-0 and
# reassigns it to mynewconsumer, but only if time since delivery
# is greater than 1second
XCLAIM mystream mygroup mynewconsumer 1000 1705335383125-0
```

Redis introduced XAUTOCLAIM to combine both XPENDING and XCLAIM,
no need to first get the id.

```bash
XAUTOCLAIM mystream mygroup mynewconsumer 1000 COUNT 1
```

Note:

- claiming the message resets the time since delivery and increments the delivery counter. The delivery counter can be set to arbitrary value with XCLAIM's RETRYCOUNT
- Any redis process can reassign the message, not just a member of the consumer group

Consumer recovery scenario:

- Monitor pending messages with XPENDING
- Determine which messages should be reassigned by choosing a heuristic
  - delivery count
  - elapsed time since deliery
  - consumer idle time...
- Decide how to reassign pending messages:
  - random
  - round-robin
  - lowest idle time consumer...

Note: A poison pill message is a bad unprocessable message that consumers can not acknowledge. Such a message can kill the consumer, cause it to be reprocessed indefinitely, etc. If such a message was to be continuously reclaimed, it could cause issues. We can keep track of how many times the message was claimed and decide on the approach to take (forcing ack, deleting it...)

### Performance

- XADD is the only way to add messages to the stream, It is O(1). XREAD, XRANGE, XDEL performance is also O(1). While it is not surprising that this is the true from the end of the stream, it works from any position
- So even if it is comfortable to think of redis as an append only log, it is actually implemented as a radix tree (basically an optimized trie). So getting something is O(M) where M is the size of the key id, which is very short so it can be seen as O(1)
- Note that when using COUNT 1000, it is O(1000) so be careful with the count argument

In addition to the radix tree, when multiple consecutive messages have the same field name, Redis will not replicate the storage of the field names and instead each subsequent message having the same set of field names is marked with a flag. As such, it can be a lot more memory efficient than other datastructures as can be seen with [MEMORY USAGE](https://redis.io/commands/memory-usage/)

To manage the stream length:

- do it directly with XADD with the MAXLEN subcommand
  - tightly controlled stream growth
  - no need for extra component as capping is done by producer
  - cons: trimming to exact size on each XADD is not the most efficient but you can use ~ to trim to approximate length for better perf. Under the hood, redis will only remove the message if it can remove a whole node from the radix tree
- periodically trimming with XTRIM
  - can be implemented by an admin or an automated component
- by minid with MINID
- by time:
  - partition the stream by date time (one new key for each date)
  - set expiration of each stream
  - make sure the consumer group reads from the correct partition

### Streams usage patterns

#### Large stream payloads

For ex 50kb messages

- use of trim
- store payload somewhere else and only reference it from the stream. If payload needs to be hot, store it in a Hash, else on disk/large object stores

#### Single stream vs multiple streams

- depends on the stream's access patterns
- For ex user notifications are better with one stream per user
- However an API access log is better suited for a single stream

- Multiple streams will naturally partition across a redis cluster
- With a large stream, we must be careful to cap it so that it does not use all the memory on a given node

#### single consumer vs consumer group

- Single consumer processes the entire stream with XREAD. We must keep track of the last processed ID manually
- With the consumer group, the last delivered message is automatically stored in the group

- Consumer groups make more sense if you need out of order processing, if you have CPU intensive workloads
- In other cases, it is better to use a single consumers. Even if you need stream segmentation, this can be accomplished with a single consumer. To do that each consumer just needs to keep track of its own offset in the stream
