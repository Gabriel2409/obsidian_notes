#dsa

## Definition

Subset of [[Two pointers]] where we look at the subarray between left and right pointer

## Examples

- Kadane algorithm: greedy/dynamic programming algorithm to calculate the maximum sum subarray ending at a particular position. Idea is to explore array from left to right and for each element, only include the previous sum if it is strictly positive. If it is negative, do not include it (which basically is like moving the left pointer to the current element)

```python
def max_subarray(nums):
    maxsum = nums[0]
    cur = 0

    for n in nums:
        cur = max(cur, 0)
        cur += n
        maxsum = max(maxsum, cur)
    return maxsum
```

- Fixed size sliding window. For ex: Given an array, return true if there are two elements within a window of size `k` that are equal. Idea is to maintain a hashset containing the previously visited element and then at each step, move the sliding window to the right (which removes the leftmost element from the set)

```python
def check_duplicates(nums, k):
    window = set(nums[0]) # Cur window of size <= k
    l = 0

    for r in range(1, len(nums)):
        if r - l + 1 > k:
            window.remove(nums[l])
            l += 1
        if nums[r] in window:
            return True
        window.add(nums[r])
    return False
```

- Variable size sliding window. For ex: Find the length of the longest subarray, with the same value in each position. Idea is to move right pointer until we find a different char then put left pointer on right pointer.

```python
def longestSubarray(nums):
    length = 0
    l = 0

    for r in range(len(nums)):
        if nums[l] != nums[r]:
            l = r
        length = max(length, r - l + 1)
    return length
```
