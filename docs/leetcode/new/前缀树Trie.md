## Trie (前缀树)

### 208. 实现 Trie (前缀树)

Trie（发音类似 "try"）或者说 前缀树 是一种树形数据结构，用于高效地存储和检索字符串数据集中的键。这一数据结构有相当多的应用情景，例如自动补完和拼写检查。

请你实现 Trie 类：

- Trie() 初始化前缀树对象。
- void insert(String word) 向前缀树中插入字符串 word 。
- boolean search(String word) 如果字符串 word 在前缀树中，返回 true（即，在检索之前已经插入）；否则，返回 false 。
- boolean startsWith(String prefix) 如果之前已经插入的字符串 word 的前缀之一为 prefix ，返回 true ；否则，返回 false 。

```java
class Trie {
    private final Node root = new Node();

    private static class Node {
        Node[] table = new Node[26];
        boolean isLeaf;
    }

    public void insert(String word) {
        Node n = root;
        for (char c : word.toCharArray()) {
            if (n.table[c - 'a'] == null)
                n.table[c - 'a'] = new Node();
            n = n.table[c - 'a'];
        }
        n.isLeaf = true;
    }

    public boolean search(String word) {
        Node n = root;
        for (char c : word.toCharArray()) {
            if (n.table[c - 'a'] == null) return false;
            n = n.table[c - 'a'];
        }
        return n.isLeaf;
    }

    public boolean startsWith(String prefix) {
        Node n = root;
        for (char c : prefix.toCharArray()) {
            if (n.table[c - 'a'] == null) return false;
            n = n.table[c - 'a'];
        }
        return true;
    }
}
```
