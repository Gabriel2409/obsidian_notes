---
sr-due: 2023-08-13
sr-interval: 61
sr-ease: 250
---

#dsa

## Implementation

- Hashmaps can be implemented using arrays under the hood.
  - When the hashmap is created, an array of given size is initialized and the memory is reserved.
  - Even when the hashmap is empty, the array still takes up memory

### Hashing

- To insert a given key-value pair in a hashmap, we first use a hash function that converts our key to an integer.
- Then we use the modulo operator to place it in the array

```text
Ex: We start with an empty hashmap. Associated array capacity is 4. We want to insert A:1 in the hashmap
A -> hashing -> 47 % 4 -> 3 -> we store A:1 at index 3 of the array
```

### Resizing

In order to ensure that the `key:value` pair ends up in a vacant spot, we keep track of how many spots were occupied in the array.
When we occupy more than half of the capacity, we resize the array.
This operation is a lot similar to what happens under the hood when resizing dynamic arrays:

- Allocate new array with double capacity
- Loop through all the elements in the hashmap, apply the hash function and the modulo operator with the new capacity, store them in the new array

Note: resizing occurs when array becomes half full which can create some performance issues if there are too many resizing.

### Collisions

When two different keys end up in the same position in the array, there is a collision.
There are two main ways to deal with collisions

#### Chaining

When chaining, each element of the array stores a linked list.
When there is a collision, the new element will be appended at the end of the list

Insertion and deletion are still O(1) in average as long as the number of elements in the linked list is small enough

```python
# example implementation of hashmap whose default value is None implemented with chaining

class Node:
    def __init__(self, key, val):
        self.key = key
        self.val = val
        self.next = None

    def __repr__(self):
        r = f"{self.key}:{self.val}"
        while self.next:
            self = self.next
            r += f"->{self.key}:{self.val}"
        return r

class Hashmap:
    def __init__(self):
        self.capacity = 4
        self.nb_inserted = 0
        self.arr: List[Optional[Node]] = [None for _ in range(self.capacity)]

    def hash(self, key: str) -> int:
        """sums ascii chars"""
        total = 0
        for char in key:
            total += ord(char)
        return total

    def put(self, key: str, val):
        ind = self.hash(key) % self.capacity

        if self.arr[ind] is None:
            self.arr[ind] = Node(key, val)
            self.nb_inserted += 1
        else:
            node = self.arr[ind]
            while True:
                if key == node.key:
                    node.val = val
                    break
                if node.next is None:
                    node.next = Node(key, val)
                    self.nb_inserted += 1
                    break
                node = node.next
        if self.nb_inserted >= self.capacity // 2:
            self.resize()

    def resize(self):
        self.capacity = self.capacity * 2
        self.nb_inserted = 0
        oldarr = self.arr.copy()
        self.arr = [None for _ in range(self.capacity)]
        for el in oldarr:
            node = el
            while node:
                self.put(node.key, node.val)
                node = node.next

    def get(self, key):
        ind = self.hash(key) % self.capacity
        if self.arr[ind] is None:
            return None
        else:
            node = self.arr[ind]
            while node:
                if node.key == key:
                    return node.val
                node = node.next
            return None

    def remove(self, key):
        ind = self.hash(key) % self.capacity
        if self.arr[ind] is None:
            # no pop
            return None
        else:
            node = self.arr[ind]
            if key == node.key:
                self.arr[ind] = node.next
                self.nb_inserted -= 1
                return node.val

            else:
                prev = node
                cur = node.next
                while cur:
                    if cur.key == key:
                        prev.next = cur.next
                        self.nb_inserted -= 1
                        return cur.val
                return None



```

#### Open addressing

- When a collision occurs, find the next empty spot in the array and store the value here (go back to beginning of array if necessary)
- On deletion, put a sentinel value where the deletion occurs so that we don't stop searching too early for previously stored values
- when resizing, discard the sentinels

```python

class Hashmap:
    def __init__(self):
        self.capacity = 4
        self.nb_inserted = 0
        self.arr = [None for _ in range(self.capacity)]

    def hash(self, key: str) -> int:
        """sums ascii chars"""
        total = 0
        for char in key:
            total += ord(char)
        return total

    def put(self, key: str, val):
        ind = self.hash(key) % self.capacity
        while self.arr[ind] is not None:
            if self.arr[ind] == -1:
                ind += 1
            elif self.arr[ind][0] == key:
                self.arr[ind] = (key, val)
                return
            else:
                ind += 1
            if ind == len(self.arr):
                ind = 0

        self.arr[ind] = (key, val)
        self.nb_inserted += 1

        if self.nb_inserted >= self.capacity // 2:
            self.resize()

    def resize(self):
        self.capacity = self.capacity * 2
        self.nb_inserted = 0
        oldarr = self.arr.copy()
        self.arr = [None for _ in range(self.capacity)]
        for el in oldarr:
            if el is not None and el != -1:
                self.put(el[0], el[1])

    def get(self, key):
        ind = self.hash(key) % self.capacity
        while self.arr[ind] is not None:
            if self.arr[ind] == -1:
                ind += 1
            elif self.arr[ind][0] == key:
                return self.arr[ind][1]
            else:
                ind += 1
            if ind == len(self.arr):
                ind = 0
        return None

    def remove(self, key):
        ind = self.hash(key) % self.capacity
        while self.arr[ind] is not None:
            if self.arr[ind] == -1:
                ind += 1
            elif self.arr[ind][0] == key:
                val = self.arr[ind][1]
                self.arr[ind] = -1
                return val
            else:
                ind += 1
            if ind == len(self.arr):
                ind = 0
        return None

```
