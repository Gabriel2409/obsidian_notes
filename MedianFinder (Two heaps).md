---
sr-due: 2023-05-11
sr-interval: 3
sr-ease: 250
---

#dsa

If we want to keep track of the median of a set of value, we can use an array. Retrieving the median is O(1) but inserting new elements is O(n).

By using two heap, a min heap keeping track of the larges values, and a max heap keeping track of the smaller values, we can reduce insertion time to O(logn)

## Implementation

### Retrieval of median

- Idea is to get the first element of the heap with the highest length or the mean of both if they have the same length.
- Note that we multiply by -1 the value of the maxHeap because we suppose it is implemented as a python minheap under the hood

```python
def get_median(minH, maxH):
    if len(minH) > len(maxH):
        return minH[0]
    elif len(minH) < len(maxH):
        return -1 * maxH[0]
    else:
        return (minH[0] - maxH[0]) / 2
```

### Insert new element

- Insert in the max heap
- check first elements of two heaps. If it is larger in max heap, pop it and add popped value to min heap.
- Balance so that there is at most a diff of 1 in the length

```python

def insert(val, minH, maxH):
    heapq.heappush(maxH, -1 * val)
    if maxH and minH and -1 * maxH[0] > minH[0]:
        popped = heapq.heappop(maxH)
        heapq.heappush(minH, -1 * popped)

        if len(minH) > len(maxH) + 1:
            popped = heapq.heappop(minH)
            heapq.heappush(maxH, -1 * popped)

        if len(maxH) > len(minH) + 1:
            popped = heapq.heappop(maxH)
            heapq.heappush(minH, -1 * popped)


```
