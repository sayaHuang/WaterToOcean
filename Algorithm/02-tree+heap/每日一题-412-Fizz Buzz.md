[TOC]

# 每日一题-Fizz Buzz

## 题目
写一个程序，输出从 1 到 n 数字的字符串表示。  
1. 如果 n 是3的倍数，输出“Fizz”；
2. 如果 n 是5的倍数，输出“Buzz”；
3.如果 n 同时是3和5的倍数，输出 “FizzBuzz”。

**示例:**  
```java
n = 15,

返回:
[
    "1",
    "2",
    "Fizz",
    "4",
    "Buzz",
    "Fizz",
    "7",
    "8",
    "Fizz",
    "Buzz",
    "11",
    "Fizz",
    "13",
    "14",
    "FizzBuzz"
]
```

### 读题所得
1. 输出字符串数组

## 方法一:自己得思路
```java
    final String Fizz = "Fizz";
    final String Buzz = "Buzz";

    List<String> ans = new ArrayList();
    public List<String> fizzBuzz(int n) {
        for (int i = 1; i <= n; ++i) {
            String result = "";
            if (i % 3 == 0) {
                result += Fizz;
            }

            if (i % 5 == 0) {
                result += Buzz;
            }
            ans.add(result.equals("") ? String.valueOf(i) : result);
        }
        return ans;
    }
```
### 复杂度
* 时间复杂度: O(N)
* 空间复杂度: O(1)

## 方法二: 参考解题
```swift
    func fizzBuzz(_ n: Int) -> [String] {
        //通过计数确认是否为 3 或 5 的倍数
        var fizz = 0
        var buzz = 0
        var result = [String]()
        for index in 1...n {
            fizz += 1
            buzz += 1
            if fizz ==  3 && buzz == 5 {
                fizz = 0
                buzz = 0
                result.append("FizzBuzz")
            } else if fizz ==  3 {
                fizz = 0
                result.append("Fizz")
            } else if buzz ==  5 {
                buzz = 0
                result.append("Buzz")
            } else {
                result.append(String(index))
            }
        }
        return result
    }
```
### 复杂度
* 时间复杂度: O(N)
* 空间复杂度: O(1)

## 测试用例
3  
5  
7  
随机数  

## leetcode链接
[leetcode链接题目链接]((https://leetcode-cn.com/problems/fizz-buzz/))  