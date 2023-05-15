---
sr-due: 2023-05-09
sr-interval: 1
sr-ease: 230
---

#dsa #todo

## Concept

Backtracking is a way to explore all possible
paths. It usually works with a depth first search algorithm:

```python
...
path.append(i)
dfs(i+1)
path.pop()
dfs(i+1)
```

## Backtracking problems

### All possible subsets

- Given an integer array of **unique** elements, return _all possible_ _subsets_ _(the power set)_.
- Idea is to include element then call depth first search on next part of the array, then pop element and do it again

```python
def subsets(nums: List[int]) -> List[List[int]]:
	final = []
	stack = []

	def dfs(i):
		if i == len(nums):
			res.append(stack.copy())
			return
		# decision to include nums[i]
		stack.append(nums[i])
		dfs(i + 1)
		# decision NOT to include nums[i]
		stack.pop()
		dfs(i + 1)
	dfs(0)
	return final
```

### Combination sum
