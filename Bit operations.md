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
