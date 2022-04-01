[TOC]

# 每日一题-第一个只出现一次的字符

## 题目
在字符串 s 中找出第一个只出现一次的字符。如果没有，返回一个单空格。
**示例:**  
```java
s = "abaccdeff"
返回 "b"

s = "" 
返回 " "
```

**限制:**
0 <= s 的长度 <= 50000  
### 读题所得
1. 处理字符串
2. 找出只出现了一次得字符
3. 如果没有返回 ' ' (空格)
4. 并没有说明字符得 码表 范围

## 方法一: Hash表
```swift
    public char firstUniqChar(String s) {
        //遍历数组, 依次以c为key保存到map中, 值: bool
        char[] chars = s.toCharArray();
        Map<Character,Boolean> reslut = new LinkedHashMap<>();
        for (int i = 0; i < chars.length; i++) {
            boolean res = reslut.containsKey(chars[i]);
            reslut.put(chars[i],!res);
        }

        for (Map.Entry<Character,Boolean> e: reslut.entrySet()) {
            if (e.getValue()) {
                return e.getKey();
            }
        }
        return ' ';
    }
```
### 复杂度
* 时间复杂度: O(N)
* 空间复杂度: O(N)

## 方法二: 参考解题
 数组保存得话, 假设为ASCII码表,码表范围从 0 ~ 127, 128 *  4B , 码表 范围过大不适合该方法
```java
final int asciiLength = 128;
public char firstUniqChar(String s) {
    char[] chars = s.toCharArray();
    int[] result = new int[asciiLength];
    for (int i = 0; i < chars.length; i++) {
        ++result[chars[i]];
    }

    //再次循环
    for (int i = 0; i < chars.length; i++) {
        if (result[chars[i]] == 1) { return chars[i]; }
    }

    return ' ';
}
```
### 复杂度
* 时间复杂度: O(N)
* 空间复杂度: O(N)

## 测试用例
""  
"l"  
"leetcode"  
"eeleetcode"  
"eel"  
"eel**"  
"ee^lth&"  
随机50000个数字

## leetcode链接
[leetcode链接题目链接](https://leetcode-cn.com/problems//)  