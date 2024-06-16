---
reviewed: 2023-07-27
sr-due: 2024-06-06
sr-interval: 3
sr-ease: 250
---

#dsa #sort

- sorting algorithm that works for integers or objects with a defined range of values.
- Counts the occurrences of each value and uses this information to place the values in sorted order

```python
def counting_sort(arr):


    max_val = max(arr)
    min_val = min(arr)

    # array to store the frequency of each element
    count = [0] * (max_val - min_val + 1)

    for num in arr:
        count[num - min_val] += 1


    sorted_arr = []
    for i, freq in enumerate(count):
        sorted_arr.extend([i + min_val] * freq)

    return sorted_arr
```

- efficient when the range of input values is not significantly larger than the number of elements to be sorted.
- time complexity: O(n + k), where n is the number of elements in the input array, and k is the range of input values.
- space complexity: O(k)
- stable
