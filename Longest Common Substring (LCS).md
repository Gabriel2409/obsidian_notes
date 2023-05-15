#dsa

## Definition

2D [[Dynamic programming]] problem to find longest common sequence in two strings (a subsequence is a subset of a given set where the relative order of the characters/elements is maintained)

- ex: `s1 = "abcdefg"`, `s2 = "bdce"` => `LCS = "bde"`

## Top down brute force

- n = len(s1), m = len(s2)
- Idea is to check the two strings at a given index.
  - If chars are equal, add 1, and increment both indices
  - Else, advance for s1 OR advance for s2 and take max

```python
def dfs(i1, i2):
    if i1 == len(s1) or i2 == len(s2):
        return 0

    # if the characters match, go to next char for both strings
    if s1[i1] == s2[i2]:
	    count = 1 + dfs(i1 + 1, i2 + 1)

    # if the characters don't match
    else:
        count = max(dfs(i1 + 1, i2), dfs(i1, i2 + 1))
    return count
```

- Time complexity: O(2^n): size of the decision tree
- Space complexity: O(n+m)

## Top down with memoization

Add a cache

```python
cache = [[-1 for _ in range(m)] for _ in range(n)]


def dfs(i1, i2):
	if cache[i1][i2] != -1:
		return cache[i1][i2]

	...
	cache[i1][i2] = count
	return count
```

- Time complexity: O(n \* m)
- Space complexity: O(n \* m)

## Bottom up

- Add dp matrix. One extra row and column compared to cache to avoid having to check for edge cases.

```python
def bottomup(s1, s2):
	dp = [[0 for _ in range(m+1)] for _ in range(n+1)]
	for i1 in range(n):
		for i2 in range(m):
			if s1[i1] == s2[i2]:
				dp[i1+1][i2+1] = 1 + dp[i1][i2]
			else:
				dp[i1+1][i2+1] = max(dp[i1+1][i2], dp[i1][i2+1])

	return dp[n][m]
```

- Time complexity: O(n \* m)
- Space complexity: O(n \* m)

## Optimized bottom up

- We can reduce space complexity to O(m) by only keeping track of the prev row

```python
def bottomup(s1, s2):
	dp = [0 for _ in range(m+1)]
	for i1 in range(n):
		cur = [0 for _ in range(m+1)]
		for i2 in range(m):
			if s1[i1] == s2[i2]:
				cur[i2+1] = 1 + dp[i1][i2]
			else:
				cur[i2+1] = max(cur[i2], dp[i2+1])
		dp = cur
	return dp[m]
```
