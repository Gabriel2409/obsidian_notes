#dsa #sort

- Method to [[Sort]] an array where each element is moved to the
  left until it encounters a smaller element.
- Time complexity: O(n^2)
- In place sorting -> Extra space complexity: O(1)
- Stable

```
[3,2,1,5,4] 2<->3
[2,3,1,5,4] 1<->3 1<->2
[1,2,3,5,4] 5 not moved
[1,2,3,5,4] 4<->5
[1,2,3,4,5]
```

```python
# method with continuous swap
def sort(arr: list[int]) -> list[int]:
    for i in range(len(arr)):
        j = i - 1
        while j >= 0 and arr[j] > arr[j + 1]:
            arr[j], arr[j + 1] = arr[j + 1], arr[j]
            j = j - 1
    return arr

# method where we keep current value in memory
def sort(arr: list[int]) -> list[int]:
    for i in range(len(arr)):
        cur = arr[i]
        j = i
        while j - 1 >= 0 and arr[j - 1] > cur:
            arr[j] = arr[j - 1]
            j = j - 1
        arr[j] = cur
    return arr

```
