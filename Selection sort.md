---
sr-due: 2023-10-02
sr-interval: 113
sr-ease: 272
---

#dsa #sort

- Method to [[Sort]] an array by finding the smallest element of the array and switching
  it with the first element.
  Then repeat the operation for all the elements and the rest of the array
- Time complexity: O(n^2)
- In place sorting -> Extra space complexity: O(1)
- Stable

```
[3,2,1,5,4] 1<->3
[1,2,3,5,4] not modified for the next 3 passes. On the last pass 5<->4
[1,2,3,4,5]
```

```python
def selection_sort(arr: list[int]):
    for i in range(len(arr)):
        minimum = i
        for j in range(i, len(arr)):
            if arr[j] < arr[minimum]:
                minimum = j
        arr[minimum], arr[i] = arr[i], arr[minimum]
```
