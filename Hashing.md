#dsa

## Definition

A hash function is any function that can be used to map data of arbitrary size to fixed-size values. The values returned by a hash function are called hash values, hash codes, digests, or simply hashes.

Hashing is the transformation of a string of characters into a usually shorter fixed-length value or key that represents the original string.

Hashing is used everywhere:
- In encryptions algorithms
- To verify the integrity of files
- In [[Hashmap]], where the key is hashed to obtain an int and this int is then mapped to a position in the underlying array (with the % operator)
- to cache database queries
- etc...

## Follow up
- [[Consistent hashing]]