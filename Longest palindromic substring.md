---
sr-due: 2023-06-19
sr-interval: 19
sr-ease: 229
reviewed: 2023-07-06
---

#dsa

## Definition

- Detecting if a word is a palindrome can be done easily with [[Two pointers]]
- Detecting the longest palindromic substring is more complex and can be done efficiently with [[Dynamic programming]]
- This problem does not fit the standard framework of implementing first a top down approach, then memoization then bottom up. However it is still dynamic programming as we break the problem into smaller chunk.

## Implementation

- Idea is to start on a given character, then explore both left and right path at the same time and stop when we encounter different characters. It must be done once for odd substrings and one for even substrings

```python
def longest_palindrome_substring(s):
	maxlen = 0
	for i in range(len(s)):
		# even substrings
		l = i
		r = i
		while s[l] == s[r] and l >= 0 and r < len(s):
			maxlen = max(maxlen, r - l + 1)
			l -= 1
			r += 1
		# same code for odd substrings (could be refactored)
		l = i
		r = i + 1 # only difference
		while s[l] == s[r] and l >= 0 and r < len(s):
			maxlen = max(maxlen, r - l + 1)
			l -= 1
			r += 1
```

- Time complexity: O(n^2)
- Space complexity: O(n)
