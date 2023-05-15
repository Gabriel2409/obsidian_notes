---
sr-due: 2023-06-05
sr-interval: 27
sr-ease: 270
---

#dsa

## Definition

Simply put, more optimized version of recursion. It takes a big problem and divides it into subproblems.
To get to the ideal solution, you can often go through the following steps:

- Top down brute force approach
- Top down memoization
- Bottom up approach
- Bottom up with space optimisation

## Illustration with fibonnaci number

### Top down Brute force

```python
def fib(n):
	if n <= 1:
		return n
	return fib(n-1) + fib(n-2)
```

- Time complexity = O(2^n)

```
			        n
			  /           \
		   n-1             n-2
		  /    \         /     \
		n-2    n-3      n-3    n-4
		:
	  1   0
Tree of height n
```

There is a lot of repeated work. Indeed fib(n-2) is called twice for ex.

### Top down Memoization

Memoization consists in caching return values so that we can return them on subsequent calls

```python
def fib(n):
	cache = {}
	if n <= 1:
		return n
	if n in cache:
		return cache[n]
	cache[n] = fib(n-1) + fib(n-2)
	return cache[n]
```

- All of the fibonnaci numbers are calculated at most once,
- Time complexity = O(n)

### Bottom up approach

By using an iterative approach, we can get rid of the recursion stack. This is usually what is actually referred as dynamic programming.

```python
def fib(n):
	dp = [[] for _ in range(n +1)]
	dp[0] = 0
	dp[1] = 1

	for i in range(2, n+1):
		dp[i] = dp[i-1] + dp[i-2]
	return dp[n]
```

- Time complexity: O(n)
- Space complexity: O(n)

### Bottom up approach with space optimisation

Usually with bottom up approach, we don't need to store all the previously calculated values.
For example, here, we only need the last two values

```python
def fib(n):
	dp = [0,1]

	for i in range(2, n+1):
		dp[0], dp[1] = dp[1], dp[0] + dp[1]
	return dp[2]
```

- Space complexity: O(1)

## Example applications

#todo

- [[Knapsack]]
