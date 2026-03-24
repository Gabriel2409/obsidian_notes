---
sr-due: 2026-03-25
sr-interval: 582
sr-ease: 230
reviewed: 2023-07-19
---

#dsa #todo

## Useful operators

AND: `&`
OR: `|`
XOR: `^`
Shift right: `>>`
Shift left: `<<`
NOT: `~`
## Bit Tricks
Check odd/even:        n & 1
Get bit at i:          (n >> i) & 1
Set bit at i:          n | (1 << i)
Clear bit at i:        n & ~(1 << i)
Toggle bit at i:       n ^ (1 << i)

Power of two check:    n & (n - 1) == 0 

Powers of 2 in binary always look like one 1 followed by all 0s:And n - 1 always flips that pattern — the 1 becomes 0, all trailing 0s become 1s so they have no bit in common

## XOR Tricks

Same numbers cancel out → a ^ a = 0
And a ^ 0 = a

Swap without temp:
a ^= b; b ^= a; a ^= b



Example — find the missing number in [0, 1, 3, 4] (2 is missing):
XOR all the indices 0^1^2^3^4 and all the values 0^1^3^4 together:
0^1^2^3^4 ^ 0^1^3^4
Everything that appears twice cancels:
0^0 = 0
1^1 = 0
3^3 = 0
4^4 = 0
2   = 2  ← only survivor
Result is 2 — the missing number.

```python
def missing(nums):
    xor = 0
    for i, n in enumerate(nums):
        xor ^= i ^ n
    return xor
```
## Counting bits

```python
def count_bits(num):
	tot = 0
	while n > 0:
		if n & 1 == 1:
			tot += 1
		n = n >> 1
	return tot
```
Time complexity: O(logn), space O(1)