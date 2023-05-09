---
sr-due: 2023-05-09
sr-interval: 1
sr-ease: 230
---

#dsa #tree

## Definition

A Trie or Prefix tree is a [[Tree]] data structure designed to find works based on a Prefix. It can do so in O(1) time.
More precisely:
Insert, search and search Prefix can be done in O(1)

```python
class TrieNode:
    def __init__(self):
        self.children = {}
        self.word = False
```

## Insertion

- Start at the root
- For each character of the word, go to the associated TrieNode or create it.
- Mark last TrieNode as word

```python
def insert(root, word):
    for c in word:
        if c not in root.children:
            root.children[c] = TrieNode()
        root = root.children[c]
    root.word = True
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
