#dsa #graph

## Definition

Union-Find is a useful tool for keeping track of nodes connected in a [[Graph]] and detecting cycles in a graph. Note that this can usually be done with dfs as well (see [[Matrix traversal]] and [[Adjacency list traversal]]) but while dfs works well with a static graph,
union find shines when we are adding edges overtime

Union find operates on disjoint sets. To perform a union on two vertices, we must ensure that they are not already connected

## Implementation

Initially, all nodes point to themselves (they are their own parent in the unionfind sense).
Each node comes with a rank which is initially set to 0

### Union

To join two nodes, we first find their highest rank ancestor and if it is different perform the join

In the example below, we join node 0 and 1:

- they have the same rank (0): we arbitrarily choose 0 as parent for 1
- Height of tree starting with 0 increases => we increase its rank by 1
  Then we join 0 and 2:
- 0 has the highest rank: it is designated as the parent. No changes in rank
  Then we join 1 and 2:
- 1 and 2 both have the same parent, nothing happens

```markdown
0 1 2 => 0 2 => 0
| / \
1 1 2
```

### Find

Find could take a long time if we did not update the parent.
For ex, below, finding the ancestor of 4 takes three steps. In order to make the find operation faster,
we implement a path compression. Each time we try to find the parent, we move the parent to point to the grandparent

```
0       0            0
|       |          /   \
1  =>   1   =>    1     2
|       |              / \
2       2             3   4
|      / \
3     3   4
|
4
```

Next time we call the path function on 4 we will get

```
     0
    /|\
   1 2 4
     |
     3
```

Note that we don't modify the rank during the find operation. As such rank is not the actual depth of the tree BUT an **upper bound**.

### Time Space Complexity

The union operation itself is O(1). However, to perform it we must call the find function.
Naively, find could be O(n) but with path compression, we can reduce it down to O(logn) as the height is divided by 2 at each call.

The union function intelligently sets the parent and the rank so that the height increases at most of 1.
By combining union and find, we get a time complexity called Inverse Ackermann, α(n), which can be simplified to constant time, O(1)).

So, if m is the number of edges we have, then the time complexity of Union-Find is O(m ∗ logn)

### Final code

```python

class UnionFind:
	def __init__(self, n):
		self.par = [i for i in range(n)]
		self.rank = [0 for _ in range(n)]

	def find(self, n):
		while p != self.par[p]:
			# path compression
			self.par[p] = self.par[self.par[p]]
			p = self.par[p]
		return p
	def union(self, n1, n2):
		p1, p2 = self.find(n1), self.find(n2)
		if p1 == p2:
			# it means n1 and n2 are already part of the same set
			return False
		if self.rank[p1] > self.rank[p2]:
			self.par[p2] = p1
		elif self.rank[p1] < self.rank[p2]:
			self.par[p1] = p2
		else:
			self.par[p2] = p1
			self.rank[p1] += 1
		return True




```
