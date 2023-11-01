---
reviewed: 2023-09-19
---

#dsa #search

- search algorithm to find target in a sorted array
- very similar to [[Binary search]] but instead of using the mid as the next position, an interpolation technique is used to estimate the position of the target based on the range of values. Moreover we must add a check to ensure the target is between `arr[low]` and `arr[hig]` to ensure that the generated pos is not out of bounds

```python
def interpolation_search(arr, target):
    low = 0  # Lower bound of the search range
    high = len(arr) - 1  # Upper bound of the search range

    while low <= high and arr[low] <= target <= arr[high]:
        # Estimate the position of the target element
        pos = low + ((target - arr[low]) * (high - low)) // (arr[high] - arr[low])

        if arr[pos] == target:
            return pos  # Element found at position 'pos'
        elif arr[pos] < target:
            low = pos + 1  # Adjust the search range to the right
        else:
            high = pos - 1  # Adjust the search range to the left

    return -1  # Element not found in the array
```

Interpolation Search is particularly efficient when the data is uniformly distributed and can have an average-case time complexity of approximately O(log log n), making it faster than binary search for such data distributions. However, it can perform poorly on non-uniformly distributed data.
