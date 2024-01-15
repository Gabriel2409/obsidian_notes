#dsa

Randomized data structure that combines elements of both [[Linked list]] and [[Binary tree]]


A skip list consists of multiple levels of linked lists, with the bottom level containing all elements in sorted order. Each higher level is a subset of the level below it, containing a fraction of the elements.

Nodes in the skip list have two pointers: `next` pointing to the next node at the same level and `down` pointing to the node at the next level. A Tower refers to a vertical sequence of nodes

The decision to include an element at a higher level is randomized, which helps balance the skip list and ensures efficient search times.

Searching in a skip list is performed in a manner similar to binary search. The search starts from the top-left corner and moves down and right, skipping over elements when possible.

Insertion and deletion involve updating pointers at multiple levels

The randomization ensures that the skip list remains balanced on average, providing logarithmic time complexity for search, insertion, and deletion operations.

Implementation: https://gist.github.com/sachinnair90/3bee2ef7dd3ff0dc5aec44ec40e2d127

```bash
[ ]------------->[ ]
[ ]->1------->8->[ ]
[ ]->1---->5->8->[ ]
[ ]->1->3->5->8->[ ]
```

