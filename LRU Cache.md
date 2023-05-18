#dsa

## Definition

A LRU Cache is a type of [[Cache]] with limited capacity that keeps only the most recently used elements (Eviction policy is to remove oldest entries when capacity is full). Here using means either adding / updating OR getting an entry.

## Implementation

A possible implemetation is by using a hashmap as the cache and a doubly linked list to keep the values in the correct order.
At the high level:

- On get:
  - retrieve the value directly from the cache (the cache points to a node and you can read the value)
  - Move the node to the end of the linked list
- On put:
  - if the key is in the cache, update the value of the pointed node and move it to the end
  - if it is not, add a new node at the end of the linked list and update the cache to point to it. If the capacity is full, move the head to the next node and remove the key associated to the evicted node from the cache

Note: In practice, we can keep a dummy head and a dummy tail to avoid having to update the pointer to head and tail.

```python
class Node:
    """Basic double linked list node."""

    def __init__(self, key, value, next=None, prev=None):
        self.key = key
        self.value = value
        self.next = next
        self.prev = prev


class LRUCache:
    def __init__(self, capacity):
        self.capacity = capacity  # max capacity of the cache
        self.filled = 0  # track nb of objects in the cache
        self.hashmap = {}  # actual cache

        # keep a dummy head and tail to avoid having to move the head and tail pointer
        # Note that we suppose that you can not pass None as a key
        self.head = Node(None, None)
        self.tail = Node(None, None)
        self.head.next = self.tail
        self.tail.prev = self.head

    def insert(self, node):
        """Inserts node at the end of the list (just before the dummy tail)"""
        beforetail = self.tail.prev
        beforetail.next = node
        self.tail.prev = node
        node.next = self.tail
        node.prev = beforetail

    def remove(self, node):
        """Removes a node. Because we use dummy tails and heads, no need for extra checks"""
        prev = node.prev
        next = node.next
        prev.next = next
        next.prev = prev

    def get(self, key):
        """Retrieves value of a key and moves it to the end of the linked list"""
        if key not in self.hashmap:
            return -1

        node = self.hashmap[key]
        self.remove(node)
        self.insert(node)
        return node.value

    def put(self, key, value):
        """Adds / updates value of a key and moves it to the end of the linked list
        If the key was not in the cache and the cache is filled, evicts the oldest entry
        """
        if key not in self.hashmap:
            node = Node(key, value)
            self.hashmap[key] = node
            self.insert(node)
            if self.filled < self.capacity:
                self.filled += 1
            else:
                del self.hashmap[self.head.next.key]
                self.remove(self.head.next)
        else:
            node = self.hashmap[key]
            node.value = value
            self.remove(node)
            self.insert(node)
```
