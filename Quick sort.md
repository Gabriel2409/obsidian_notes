---
sr-due: 2023-09-18
sr-interval: 81
sr-ease: 252
reviewed: 2023-10-07
---

#dsa #sort

Quick sort is an algorithm to [[Sort]] an array:

1. Choose a pivot (for ex the last element of the array)
2. Move the elements smaller than the pivot on the left of the array 
   - keep a pointer on first element and start looping from here while ignoring the pivot
   - each time you encounter an element smaller or equal to pivot, switch it with pointed element and increment the pointer.
   - Once the loop is finished, switch pointed element with pivot
3. Sort the left array and the right array
4. Repeat

```txt
[1,6,5,4,2,3] : pivot is 3
1 smaller than 3, we switch 1 with 1 (first element).
Then nothing until we encounter 2, we switch 2 with 6 (second element).
Loop over, we switch 3 with 5 (third element)
We get [1,2,3,4,6,5]

Now we do the same on [1,2] and [4,6,5]

[1,2] 1 smaller than 2: switch 1 with 1 (first element).
Loop is over, switch 2 with 2 (second element)
We get [1,2]

Then we do the same for [1] => no changes


[4,6,5] 4 smaller than 5, switch 4 with 4 (first element). No modif for 6.
Loop is over, switch 5 with 6 (second element)
We get [4,5,6]

Then we do the same for [4] and [6] => no changes

Final array is sorted

```

- Average time complexity: O(nlog n). Worst cast is if the pivot is always the smallest / greatest element => O(n^2)
- In place sort: Extra space complexity: O(1)
- Not stable: it exchanges non adjascent elements

```python
def qsort(arr):
    l = 0
    r = len(arr) - 1
    return helper(arr, l, r)


def helper(arr, l, r):
    if l >= r:
        return
    pvt_index = r

    i = l
    for j in range(i, r + 1):
        if j == pvt_index:
            continue
        if arr[j] <= arr[pvt_index]:
            arr[j], arr[i] = arr[i], arr[j]
            i = i + 1
    arr[pvt_index], arr[i] = arr[i], arr[pvt_index]
    helper(arr, l, i - 1)
    helper(arr, i + 1, r)
```
