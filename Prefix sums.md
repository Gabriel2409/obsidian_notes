---
sr-due: 2023-06-19
sr-interval: 20
sr-ease: 226
---

#dsa

## Definition

- Used on an array, prefix sum at the `ith` index denotes the running sum
- Example: `[1,3,8,2]` => `[1,4,12,14]`
- This can be used to get the sum of any subarray

```python
class PrefixSum:
	def __init__(self, arr):
		self.pref = []
		total = 0
		for el in arr:
			total += el
			self.pref.append(total)
	def range_sum(self, l, r):
		if l == 0:
			return self.pref[r]
		else:
			return self.pref[r] - self.pref[l - 1]
```

- Time complexity: Builiding the prefix sum: O(n) / Retrieving the sum of any subarray: O(1)

Note: In case the array is updated a lot, a [[Segment Tree]] might be a better alternative
