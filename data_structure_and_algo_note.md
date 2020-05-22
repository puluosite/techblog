# data structure
1. binary search/lower bound/uppper bound
2. tries: use map/default_map to store the prefix
```python
class TrieNode(object):
    def __init__(self):
        self.is_word = False
        self.children = {}

class Trie(object):
    def __init__(self):
        self.root = TrieNode()
    
    def add(self, word):
        """
        Add `word` to trie
        """
        node = self.root
        for char in word:
            if char in node.children:
                node = node.children[char]
            else:
                node.children[char] = TrieNode()
                node = node.children[char]
        node.is_word = True
    
    def exists(self, word):
        """
        Check if word exists in trie
        """
        node = self.root
        for char in word:
            if char not in node.children:
                return False
            node = node.children[char]
        return node.is_word
    ```
