[TOC]

# 每日一题-Pow(x, n)

## 题目
实现 pow(x, n) ，即计算 x 的 n 次幂函数
**示例:**  
```java
输入: 2.00000, 10
输出: 1024.00000

输入: 2.00000, -2
输出: 0.25000
解释: 2-2 = 1/22 = 1/4 = 0.25
```

**提示:**
1. -100.0 < x < 100.0
2. n 是 32 位有符号整数，其数值范围是 [−231, 231 − 1] 。

### 读题所得
1. 计算x的n次幂
2. -100.0 < x < 100.0
3. n 的范围 [−231, 231 − 1]

## 方法一:递归
    //使用分治的思想, 2*2*2*2*2*2 = 2^6 = 2^3*2^3
    //需要考虑 n 的奇偶性, 需要考虑n的正负数
    //需要注意n = 0 的情况
```swift
    func myPow(_ x: Double, _ n: Int) -> Double {
        var tempX = x; var tempN = n
        //对n的正负处理
        if n < 0 {
            tempX = 1.0 / tempX
            tempN = abs(n)
        }
        return myPowDfs(tempX, tempN)
    }
    func myPowDfs(_ x: Double, _ n: Int) -> Double {
        //对n == 0的处理
        if n <= 0 {
            return 1
        }
        let subResult = myPowDfs(x, n / 2)
        //对n的奇偶性的处理
        return n % 2 == 0 ? subResult * subResult : subResult * subResult * x
    }
```
### 复杂度
* 时间复杂度: O(logN)
* 空间复杂度: O(logN)

## 方法二: 迭代
x = 2  
如果n = 8  
1. n=8,  tempx = 4,  result = 0  
2. n=4, tempx = 16, result = 0  
3. n=2,  tempx = 256, result = 0  
4. n=1,  result = 256  

如果n = 5  
1. n=5,  tempx = 4, result = 2  
2. n=2, tempx = 16, result = 2
3. n=1,  result = 32
```swift
    //迭代的方式
    func myPow(_ x: Double, _ n: Int) -> Double {
        var tempX = x; var tempN = n;
        //处理n的正负情况
        if n < 0 {
            tempX = 1 / tempX
            tempN = abs(tempN)
        }
        //处理n == 0的情况
        var result :Double = 1.0
        while tempN != 0 {
            //处理n的奇偶情况
            if tempN % 2 == 1 {
                result *= tempX
            }
            tempX *= tempX
            tempN /= 2
        }
        return result
    }
```
### 复杂度
* 时间复杂度: O(logN)
* 空间复杂度: O(1)

## 测试用例
2.00000, -2  
2.00000, 10  
2.00000,  0  
-2.00000, -2  
-2.00000, 10  
0, 0  

## leetcode链接
[leetcode链接题目链接](https://leetcode-cn.com/problems/powx-n/) 