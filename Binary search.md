---
sr-due: 2023-06-12
sr-interval: 35
sr-ease: 270
---

#dsa #search

## Definition

Binary search allows to search for a given value in a sorted array by eliminating
half of the possibilities at each step. This leads to a time complexity of O(logn).

### Recursive implementation

- Idea is to recursively call a function which looks at half of the remaining array

```python
def binary_search_rec(arr, target):
    return helper(arr, 0, len(arr), target)

def helper(arr, l, r, target):
    if l == r:
        return -1
    mid = (l + r) // 2
    if arr[mid] < target:
        return helper(arr, mid + 1, r, target)
    elif arr[mid] > target:
        return helper(arr, l, mid, target)
    else:
        return mid # index of value to find
```

Note: Starting r at len(arr) instead of the last index makes it behave similarly as if
we recursively passed arr[l:r]

### Iterative implementation

- Same idea but with a while loop.
  This time, we started r on the last index because it does not matter

```python
def binary_search(arr: list[int], target):
    l = 0
    r = len(arr) - 1

    while l <= r:
        mid = l + (r - l) // 2  # avoids overflow, equivalent to (l+r)//2

        if arr[mid] < target:
            l = mid + 1
        elif arr[mid] > target:
            r = mid - 1
        else:
            return mid # index of value to find
    return -1
```
