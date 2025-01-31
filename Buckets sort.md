---
reviewed: 2023-07-27
sr-due: 2024-11-26
sr-interval: 99
sr-ease: 250
---

#dsa #sort

- sorting algorithm that distributes elements of an array into a number of buckets.
- Each bucket is then sorted individually, either using a different sorting algorithm or by recursively applying the bucket sort algorithm.
- Finally, the sorted elements from all buckets are concatenated to produce the final sorted array.

```python
def bucket_sort(arr):

    min_val = min(arr)
    max_val = max(arr)


    bucket_range = max_val - min_val
    bucket_size = bucket_range / len(arr)

    # Create empty buckets
    buckets = [[] for _ in range(len(arr))]

    # Place each element into its corresponding bucket
    for num in arr:
        index = int((num - min_val) / bucket_size)
        buckets[index].append(num)

    # Sort each bucket (you can use any sorting algorithm here)
    for bucket in buckets:
        bucket.sort()

    # Concatenate sorted buckets to produce the final sorted array
    sorted_arr = [num for bucket in buckets for num in bucket]

    return sorted_arr

```

- most effective when the input data is uniformly distributed across a range of values.
- time comp: O(n) + k\* O(sortbucket)
- space comp: O(n+ k + sortbucket)
- Stable if bucket sorting algorithm is stable
