[TOC]

# 每日一题-实现 Trie (前缀树)

## 题目
实现一个 Trie (前缀树)，包含 insert, search, 和 startsWith 这三个操作。  
**示例:**  
```java
Trie trie = new Trie();

trie.insert("apple");
trie.search("apple");   // 返回 true
trie.search("app");     // 返回 false
trie.startsWith("app"); // 返回 true
trie.insert("app");   
trie.search("app");     // 返回 true
```

**提示:**
1. 你可以假设所有的输入都是由小写字母 a-z 构成的。
2. 保证所有输入均为非空字符串。

### 读题所得

## 方法一:
```java
class Trie {
    TrieNode root;

    /** Initialize your data structure here. */
    public Trie() {
        root = new TrieNode();
    }

    /** Inserts a word into the trie. */
    public void insert(String word) {
        TrieNode node = root;
        for (int i = 0; i < word.length(); i++) {
            char c = word.charAt(i);
            if (!node.containsKey(c)) {
                node.put(c,new TrieNode());
            }
            node = node.get(c);
        }
        //设置结束
        node.setEnd(true);
    }

    /** Returns if the word is in the trie. */
    public boolean search(String word) {
        TrieNode node = prefixWith(word);
        return node != null && node.isEnd;
    }

    /** Returns if there is any word in the trie that starts with the given prefix. */
    public boolean startsWith(String prefix) {
        return prefixWith(prefix) != null;
    }

    private TrieNode prefixWith(String prefix) {
        TrieNode node = root;
        for (int i = 0; i < prefix.length(); i++) {
            char c = prefix.charAt(i);
            if (node.containsKey(c)) {
                node = node.get(c);
            } else {
                return null;
            }
        }
        return node;
    }
}

class TrieNode {
    /**保存结点*/
    TrieNode[] links;
    /**标识是否结束*/
    boolean isEnd;
    /**由于只有小写字母*/
    int count = 26;

    public TrieNode() {
        links = new TrieNode[count];
        isEnd = false;
    }

    /**判断是否包含该字段*/
    public boolean containsKey(char c) {
        return links[c-'a'] != null;
    }

    /**设置*/
    public void put(char c,TrieNode node) {
        links[c-'a'] = node;
    }

    /**获取*/
    public TrieNode get(char c){
        return links[c-'a'];
    }

    /**设置结束*/
    public void setEnd(boolean isEnd) {
        this.isEnd = isEnd;
    }

    /**获取是否结束*/
    public boolean isEnd() {
        return isEnd;
    }
}
```
### 复杂度
* 时间复杂度: O(N)  N为字符串长度
* 空间复杂度: O(26^N^) N为最长字符串长度

## leetcode链接
[leetcode链接题目链接](https://leetcode-cn.com/problems/implement-trie-prefix-tree)  