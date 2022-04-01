[TOC]

# Trie树

## 什么是Trie树
是一种专门处理`字符串匹配`的树形数据结构, 用来解决一组字符串集合中快速查找某个字符串的问题.   
Trie树的本质就是利用集合中字符串的`公共前缀`,将重复的前缀`合并`在一起.  
假如有 { how, hi, the, to, too }   
```C
               root
            /        \
           /          \
          /            \
         h              t
        /  \          /   \  
       i(.) o        h     o (isEnd=true)
             \      /       \
              w(.) e(.)   o(isEnd=true)
//(.) 为省略版本的 (isEnd=true) , 位置不够了 
```
其中每个结点标识一个字符串中的字符, 从根节点到被`isEnd == true`标识的结点的一条路径表示一个字符串  

## 为什么有Trie树
`不需要精确匹配, 在字符集中查找字符串的时候, 仅需要匹配前缀`, 适合各种补全功能  

## Trie的代码实现
### 存储Trie树
Tire树的设计参考了`hash`的思维, 将数组的下标同字符串中的单个字符一一对应.   
假如字符串只包含 [a,z] 则 { how, hi, the, to, too } 这个例子中需要 26 + 26 * 26 + 26 * 26 * 26, 这个是个等比数列的和,所以字符串越长, 消耗的空间成指数增长O(26^n^), 这么多个 存储 指针的空间(在java 64位的操作系统中, 每个对象的指针的大小为16B)  

### 代码实现
```java
class Trie {
    TrieNode root;
    public Trie() {
        //第一层不包含任何数据
        root = new TrieNode();
    }
    
    public void insert(String word) {
        TrieNode head = root;
        char[] chars = word.toCharArray();
        for (int i = 0; i < chars.length; ++i) {
            if (!head.containsChar(chars[i])) {
                head.set(chars[i],new TrieNode());
            }
            head = head.next(chars[i]);
        }
        head.setEnd(true);
    }
    
    public boolean search(String word) {
        TrieNode node = perfixWith(word);
        return node != null && node.end();        
    }
    
    public boolean startsWith(String prefix) {
        TrieNode node = perfixWith(prefix);
        return node != null;
    }

    private TrieNode perfixWith(String prefix) {
        TrieNode head = root;
        char[] chars = prefix.toCharArray();
        for (int i = 0; i < chars.length; ++i) {
            if (head.containsChar(chars[i])) {
                head = head.next(chars[i]);
            } else {
                return null;
            }
        }
        return head;
    }
}
class TrieNode {

    //index 标识 字符, 如果对应的TrieNode不为空, 则表示包含这个字符,见 containsChar 方法
    //value标识该字符链接的下一个字符的TrieNode指针
    TrieNode[] container;
    boolean isEnd;
    TrieNode() {
        isEnd = false;
        container = new TrieNode[26];
    }
    //是否包含某个char
    public boolean containsChar(char c) {
        return container[c-'a'] != null;
    }

    //获取下一个节点
    public TrieNode get(char c) {
        return container[c-'a'];
    }   

    //设置
    public void set(char c, TrieNode node) {
        container[c-'a'] = node;
    }

    //前往下一个节点
    public TrieNode next(char c) {
        return container[c-'a'];
    } 

    //设置结束
    public void setEnd(boolean isEnd) {
        this.isEnd = isEnd;
    }

    //是否结束
    public boolean end() {
        return isEnd;
    }
}
```

### 时间复杂度
**构建** 所有字符的长度如果为N, 则时间复杂度为O(N)  
**插入** 如果查询的字符串长度为L, 则时间复杂度为O(L)  
**删除**   
**查询** 如果查询的字符串长度为L, 则时间复杂度为O(L)  


## 为什么选择Trie树
其实 `散列表`  `红黑树` `跳表`也都可以在数据是动态的情况下, 快速的查找某个字符串. 但是为什么会选择Trie树呢  
**Trie树 优势**  
1. 如果需要频繁的查询某些字符, Trie树的效率会很高, 如果字符长度为L, 则时间复杂度为O(L)

**Trie树 劣势**  
1. 如果集合中数据公共前缀并不多的情况下, 就会浪费很多内存, 雪上加霜的是, 如果字符串并不止26个字符呢  
  * 有俩个因素会影响到存储空间的大小 
  * `字符串中可能包含的字符数量` 
  * `集合中最长的字符串的长度
2. 由于Trie树使用的是指针, 所以对CPU缓存不是很友好

**什么情况下使用Trie树**  
1. 字符集长度不大的情况下,Trie树的空间消耗在可接受的情况下
2. 字符集中公共前缀重合比较多的情况下, 也是为了减少空间消耗
3. `不需要精确匹配, 在字符集中查找字符串的时候, 仅需要匹配前缀`

**如何解决劣势**  
**方法一**字符的长度来源于用户, 所以我们可以从`字符串中可能包含的字符数量` 这点来入手, 可以考虑将数组更换为支持快速插入和查找的动态数据结构: 跳表, 红黑树, 散列表  
假如更换为跳表:   
  **插入**那么插入的时候每个节点可以按照字母在码表中的大小顺序来插入, 并且插入一个字符的速度为O(logN), 一个字符串的速度为O(N * logN), N为字符串的长度,   
  **查询** 类似插入可以知道查询的时间复杂度也为O(N * logN)  
  **空间复杂度** 每一层不再是固定的消耗26个指针节点的空间了, 而是可以支持动态的扩容  

**方法二** 缩点优化. 对只有一个子节点的节点，而且此节点不是一个串的结束节点，可以将此节点与子节点合并  