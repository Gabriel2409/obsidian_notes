---
sr-due: 2023-05-07
sr-interval: 3
sr-ease: 252
---

#dsa #sort

- Method to [[Sort]] an array where we first divide each part of the array
  in two until there is only one element before we merge them back together.
  The merge part occurrs on already sorted arrays, which means it can be done in one pass with two pointers
- Time complexity: O(nlogn): We can visualize it as cutting the array in half each time (log n cuts in O(1))
  and then merging all the array of approximately the same length together
  (log n merges in O(n) because we need to go through all the elements).
  Note that in practice, all the merges don't occur at the same time
- Not in place: extra space complexity: O(n) because we store left and right part to build final array
- Stable

```
        [4,2,1,3,5]
   [4,2,1]      [3,5]
 [4,2]  [1]       .
[4] [2]  .        .
 [2,4]   .        .
   [1,2,4]        .
      .        [3] [5]
      .         [3,5]
        [1,2,3,4,5]
```

```python
def msort(arr):
    if len(arr) <= 1:
        return arr
    mid = len(arr) // 2
    left = msort(arr[:mid])
    right = msort(arr[mid:])
    final = merge(left, right)
    return final

def merge(left, right):
    i = 0
    j = 0
    final = []
    while i < len(left) or j < len(right):
        if i == len(left):
            final.append(right[j])
            j = j + 1
        elif j == len(right):
            final.append(left[i])
            i = i + 1
        else:
            if left[i] <= right[j]:
                final.append(left[i])
                i = i + 1
            else:
                final.append(right[j])
                j = j + 1
    return final
```

Note: can also be implemented by keeping track of the indices but you still need to
copy the left and right part before the merge so it can not be used to reduce the memory
needed.
