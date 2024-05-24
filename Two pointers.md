---
sr-due: 2023-08-22
sr-interval: 55
sr-ease: 234
reviewed: 2023-07-18
---

#dsa

## Definition

Two pointers is a technique used on array problems where there is a left pointer `l` and a right pointer `r`, both starting at some index of the array. This usually allows a decrease in time complexity compared to brute force approaches

## Examples

- Check if a string is a palindrome (can be read in both directions)

```python
def ispali(mystring):
	l = 0
	r = len(mystring) - 1

	while l < r:
		if mystring[l] != mystring[r]:
			return False
		l += 1
		r -= 1
	return True
```

- [[Binary search]]
- [[Sliding window]] is a subset of Two pointers
