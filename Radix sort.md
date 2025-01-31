---
reviewed: 2023-10-12
sr-due: 2024-10-26
sr-interval: 46
sr-ease: 230
---

#dsa #sort

- Non-comparative sorting algorithm which does not rely on the comparison of elements to determine their order, but exploits the positional properties of the numbers being sorted.
- Linear Time Complexity: O(nk), where "n" is the number of elements in the array, and "k" is the average number of digits in the numbers.
- Stable
- Digit-by-Digit Sort: sorts starting from the least significant digit (LSD) to the most significant digit (MSD) or vice versa. Each iteration requires a linear stable sorting algorithm such as [[Buckets sort]] or [[Counting sort]]
- Suitable for Fixed-Length Data:
- Memory Usage: requires extra memory to hold the buckets during the sorting process: O(n+k)

```python

def radix_sort(arr):
    max_num = max(arr)

    exp = 1
    while max_num // exp > 0:
        # can use bucket sort as well
        counting_sort(arr, exp)
        exp *= 10
```

Note that the counting sort used is a variant that only sorts by the specified digit

```python
def counting_sort(arr, exp):
    n = len(arr)
    output = [0] * n
    count = [0] * 10

    # Count occurrences of digits at
    # the given exp position
    for i in range(n):
        index = arr[i] // exp
        count[index % 10] += 1

    # Update count[i] to store the actual
    # position of this digit in output
    for i in range(1, 10):
        count[i] += count[i - 1]

    # Build the output array
    i = n - 1
    while i >= 0:
        index = arr[i] // exp
        output[count[index % 10] - 1] = arr[i]
        count[index % 10] -= 1
        i -= 1

    # Copy the output array to the
    # original array
    for i in range(n):
        arr[i] = output[i]

```
