---
sr-due: 2023-06-10
sr-interval: 19
sr-ease: 230
reviewed: 2023-07-19
---

#dsa #tree

## Definition

A Trie or Prefix tree is a [[Tree]] data structure designed to find words based on a Prefix. It can do so in O(1) time.
More precisely:
Insert, search and search Prefix can be done in O(1)

```python
class TrieNode:
    def __init__(self):
        self.children = {}
        self.word = False
        self.refs = 0 # tracks the nb of words
```

## Insertion

- Start at the root
- For each character of the word, go to the associated TrieNode or create it.
- Mark last TrieNode as word

```python
def insert(root, word):
    root.refs += 1
    for c in word:
        if c not in root.children:
            root.children[c] = TrieNode()
        root = root.children[c]
        root.refs += 1
    root.word = True
```

## Delete

To delete a word, we just have to remove it and decrease the refs of the parents.
This allows to stop the process when the refs becomes 0.
Below implementation only works if we can guarantee word is in the Trie.

```python
def delete(root, word):
    root.refs -= 1
    if root.refs == 0:
        root.children = {}
        root.word = False
        return
    for c in word:
        child = root.children[c]
        child.refs -= 1
        if child.refs == 0:
            root.children[c] = None
            return
        root = child

    root.word = False
```

## Search

- In a Trie, it is possible to search if a word exists and also if there are word that Start with a given Prefix. Fonctions are quasi identical

```python
def search_word(root, word):
    for c in word:
        if not root.children[c]:
            return False
    return root.word
```

For search_Prefix, replace last line with `return True`
