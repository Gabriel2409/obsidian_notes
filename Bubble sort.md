---
sr-due: 2023-06-06
sr-interval: 28
sr-ease: 272
---

#dsa #sort

- Method to [[Sort]] an array by switching adjascent elements to ensure the greater
  one is on the right. It requires multiple passes.
  Each pass moves the remaining greatest value on the right
- Time complexity: O(n^2)
- In place sorting -> Extra space complexity: O(1)
- Stable

```
[3,2,1,5,4] 3<->2 3<->1 5<->4
[2,1,3,4,5] 2<->1
[1,2,3,4,5] next passes don't modify array
```

```python
def bubble_sort(arr: list[int]):
    for i in range(len(arr)):
        for j in range(len(arr) - i - 1):
            if arr[j] > arr[j + 1]:
                arr[j + 1], arr[j] = arr[j], arr[j + 1]
```
