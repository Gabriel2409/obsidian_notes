---
sr-due: 2023-09-02
sr-interval: 62
sr-ease: 234
reviewed: 2023-07-08
---

#dsa

## Definition

2D [[Dynamic programming]] problem:
Given a bag of capacity C and a list of items (profit / weight), return the max profit such that total weight is below capacity.

- 0/1 version: Each item can only be used once
- Unbounded version: items can be used several times

## Top down brute force

```python
def dfs(i, capacity):
    if i == len(profit):
        return 0
    # value when not including current item
    total = dfs(i+1, capacity)
    if weight[i] <= capacity:
        # value when including current item
        with_current = profit[i] + dfs(i+1, capacity - weight[i])
        total = max(total, with_current)
    return total
dfs(0, C)
```

- At each step, we decide whether to including current item and decrease the remaining capacity or not including it. Then we move to next item
- Note: for Unbounded, replace `dfs(i+1, capacity - weight[i])` with `dfs(i, capacity - weight[i])` because when we decide to include an item, we don't automatically move to the next item
- Time complexity = O(2^n), space = O(n)

## Top down with memoization

- By caching the result, we can decrease Time complexity.

```python
n = len(profit)
cache = [[-1 for _ in range(C+1)] for _ in range(n)]
```

Then in dfs function, add:

```python
# check cache at beginning
if cache[i][capacity] != -1:
    return cache[i][capacity]

...

# fill cache at the end
cache[i][capacity] = total
return total

```

- Time complexity: O(n\* C)
- Space complexity: O(n \* C)

## Bottom up

```python
def bottomup():
    dp = [[0 for _ in range(C+1)] for _ in range(n)]
    # fill first row
    for cap in range(C+1):
        if cap >= weight[0]:
            dp[0][cap] = profit[0]

    for i in range(1, n):
        for cap in range(C+1):
            # skip previous
            total = dp[i-1][cap]
            if weight[i] <= cap:
                # included previous
                with_current = profit[i] + dp[i-1][cap - weight[i]]
                total = max(total, with_current)
            dp[i][cap] = total
    return dp[n-1][C]
```

- when we are on item i, we either skip it and then the profit is equal to the previous one, or we included it and the profit is the sum of profit of current item and profit of previous item with a lower capacity
- for unbounded, replace `profit[i] + dp[i-1][cap - weight[i]]` with `profit[i] + dp[i][cap - weight[i]]`
- Time complexity = O(n \* C)
- Space complexity = O(n \* C)

## Bottom up optimal

- We can decrease Space complexity by only keeping the previous row. In that case, Space complexity is O(C)

```python
def bottomup():
    dp = [0 for _ in range(C+1)]
    # fill first row
    for cap in range(C+1):
        if cap >= weight[0]:
            dp[cap] = profit[0]
    for i in range(1, n):
        cur = [0] * (C+1)
        for cap in range(C+1):
            total = dp[cap]
            if weight[i] <= cap:
                with_current = profit[i] + dp[cap - weight[i]]
                total = max(total, with_current)
            cur[cap] = total
        dp = cur
    return dp[C]
```

- For Unbounded, replace `profit[i] + dp[cap - weight[i]]` with `profit[i] + cur[cap - weight[i]]`
