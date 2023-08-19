---
sr-due: 2023-06-17
sr-interval: 25
sr-ease: 252
reviewed: 2023-07-11
---

#dsa #tree #todo

## Definition

Red black trees are [[BST]] that maintain a loose balance by coloring the nodes and rotating some nodes on insertion / deletion
Contrary to [[AVL Tree]] they are not strictly balanced but they still allow look ups, insertions and deletion in O(logn) time.
Code here: https://github.com/msambol/youtube/blob/master/trees/red_black_tree.py => TODO: implement

```python
class RedBlackNode:
   def __init__(self, val):
      self.val = val
      self.parent = None  # diff with standard BST: we keep track of parent and color
      self.is_red = False
      self.left = None
      self.right = None
```

Red black trees are better for use cases with a lot of insertions / deletions as it requires less rotations than AVL trees.
However for use cases with a lot of look ups and few insertions / deletions, AVL trees are better because they are strictly balanced.

## Properties

1. **Red/Black Property**: Each node is colored either red or black (in this note, red nodes will be between brackets [])
2. **Root Property:** Root is black
3. **Leaf Property:** Each null node is considered black
4. **Red Property:** If a node is red, its children are black
5. **Depth Property:** The number of black nodes from a given node to any leaf is identical

Last two constraints guarantee that the longest path from a node to a leaf (alternating red black) is at most twice as long as the shortest path (all black)

## Rotations

Operations are identical to AVL trees but we don't have to track the height.

## Insertion

Insert the node as in a standard BST as a red node. This may violate contraint 4. To fix it, apply insert_fixup on node.
Note: Only case 3a necessitates going up the tree and repeating the operation. Other cases stop the insertion algorithm

### Insertion Fix up algorithm

Fix up algorithm guarantees a maximum of 2 rotations to fix the red black tree. Indeed, rotations occur only in case 3b which stops the fix up.

#### Case 1: node is root

color it black to fix **Root property**

```
1. [N] => N
```

#### Case 2 : parent is black

No constraint is violated, do nothing

```
2.  P
     \
     [N]

```

#### Case 3: parent is red.

It means that parent is not root and therefore node has an uncle (which can be null). Moreover it guarantees grand parent is black

##### 3a: Uncle is red

recolor G U and P and apply same function on grand parent

```text

3a.     G         [G]
       / \        / \
     [U] [P] =>  U   P  => apply insert_fixup on G
 		   \          \
    	   [N]        [N]

```

##### 3b: Uncle is black

Note this can not occur on the first inserted node but it can happen after case 3a was encountered and we have to apply the fix on nodes higher in the tree

###### 3b1: Grand parent, parent and node form a line

Do a rotation on grand parent opposite to the line then recolor parent and grand parent

```text
3b1.
     |               |                  |
     G              [P]                 P
    /  \     =>     /  \      =>       /  \
   U   [P]         G   [N]           [G]  [N]
         \        /                 /
        [N]      U                 U

```

###### 3b2: Grand parent, parent and node form a triangle

Do a rotation on parent same side as the line then perform 3b1

```text
3b2.
     |               |
     G               G
    /  \     =>     /  \      =>      apply 3b1. on P
   U   [P]         U   [N]
        /                \
      [N]                [P]


```

## Deletion

Apply deletion as in a standard BST. If the deleted node was BLACK (and only then), perform the deletion fix up.

### Identify node for fix up

After deleting a black node, the balance is off by one. To fix this, we apply the fix up algorithm on a chosen node.
This node can be null but we still need reference to the parent.
Note: the initial fix up can not occur on root as we perform it on a child of the deleted node.

#### Case 1: node to delete does not have two children

```text
P         P
 \         \   => perform fix up on N
  D  =>     N
   \
    N

```

#### Case 2: node to delete has two children

Here D value is exchanged with M value and M is removed (M is min node starting from Y )
Because M has no left child (see deletion of BST), we are back to case 1 and perform the fix up on M right child

```text
   P
    \
     D
    / \   => case 1, with N as M right child and M as D
   X   Y
      /
     M
```

### Deletion Fix up algorithm

Similarly to insertion, Fix up algorithm guarantees a maximum of 2 rotations to fix the red black tree.
Note: Only case 2c1 necessitates going up the tree and repeating the operation. Other cases stop the deletion algorithm

#### Case 0: node is red

Color node black and STOP

#### Case 1: Sibling is red

It guarantees parent is black. and children of sibling are also black.

- Do a rotation on parent (if it is right child of parent => left rotation, else right rotation)
- Set parent to red and sibling to black. Then update sibling and proceed to case 2

```text
     |             |            |
     P            [S]           S
    / \    =>     / \   =>     / \  => L is new sibling, continue with case 2
   N  [S]        P   R       [P]   R
      / \       / \          / \
     L   R     N   L        N   L
```

#### Case 2: Sibling is black

##### Case 2a: Both sibling children are black

- Color sibling to red (fixes black balance at P level)
- If P was red, color it black (fixes balance at P.parent level) and STOP
- If P was black, balance can not be fixed at this step => apply fix up algorithm on P

```text
        |                |
        P?               P
      /   \    =>       / \   => if P was already black, balance is not fixed at P.parent level
	 N     S           N  [S]     Apply fix up on P.
	      / \             / \
	     L   R           L   R

```

#### Case 2c: At least one child is red

##### Case 2c1: child on the right is black (for right sibling) OR child on the left is black (for left sibling)

- Perform rotation on sibling (right rotation if it is on the right of parent and left otherwise)
- Recolor left child to black and sibling to red
- Proceed with case 2c2

```text
        |                 |              |
        P?                P?             P?
      /   \     =>       / \      =>    / \
	 N     S            N   [L]        N   L   => L is the new S and has a red right child => proceed with 2c2
	      / \                \              \
	     [L] R                S             [S]
	                           \              \
	                            R              R
```

##### Case 2c1: child on the right is red (for right sibling) OR child on the left is red (for left sibling)

- Perform rotation on parent (left rotation if sibling is on the right of parent): If fixes balance at P level because balance of N was balance of L
- Color P and R to black. Color S to original P color. Balance of R == balance of L + 1. Because P is black, balanced is fixed at S level and at S.parent level
- Note: if S becomes root, color it to black to fix root property.

```text
        |                 |             |
        P?                S             S?
      /   \     =>       / \      =>   /  \
	 N     S            P?  [R]       P    R
	      / \          / \           / \
	     L? [R]       N   L?        N  L?

```

```

```
