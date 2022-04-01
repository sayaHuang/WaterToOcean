[TOC]

# 每日一题-用两个栈实现队列

## 题目
用两个栈实现一个队列。队列的声明如下，请实现它的两个函数 appendTail 和 deleteHead ，分别完成在队列尾部插入整数和在队列头部删除整数的功能。(若队列中没有元素，deleteHead 操作返回 -1 )
**示例:**  
```java
输入：
["CQueue","appendTail","deleteHead","deleteHead"]
[[],[3],[],[]]
输出：[null,null,3,-1]  
```

**提示:**
1 <= values <= 10000  
最多会对 appendTail、deleteHead 进行 10000 次调用  

### 读题所得
1. 两个栈实现一个队列
2. 若队列中没有元素，deleteHead 操作返回 -1

## 方法一:自己得思路
```java
class CQueue {
    Stack<Integer> stackAppend;
    Stack<Integer> stackDelete;
    int size;
    public CQueue() {
        size = 0;
        stackAppend = new Stack<>();
        stackDelete = new Stack<>();
    }
    
    public void appendTail(int value) {
        int count = stackDelete.size();
        for (int i = 0; i < count; ++i) {
            stackAppend.push(stackDelete.pop());
        }
        stackAppend.push(value);
        ++size;
    }
    
    public int deleteHead() {
        if (isEmpty()) {
            return -1;
        }
        int count = stackAppend.size();
        for (int i = 0; i < count; ++i) {
            stackDelete.push(stackAppend.pop());
        }
        --size;
        return stackDelete.pop();
    }

    private boolean isEmpty(){
        return size == 0;
    }
}
```
### 复杂度
* 时间复杂度:deleteHead 和 appendTail 平均时间复杂度 O(N)
* 空间复杂度: O(N)

## 方法二: 优化
```java
class CQueue {
    Stack<Integer> stackAppend;
    Stack<Integer> stackDelete;
    public CQueue() {
        stackAppend = new Stack<>();
        stackDelete = new Stack<>();
    }
    
    public void appendTail(int value) {
        stackAppend.push(value);
    }
    
    public int deleteHead() {
        if (!stackDelete.isEmpty()) {
            return stackDelete.pop();
        }
        if (stackAppend.isEmpty()) {
            return -1;
        } else {
            int count = stackAppend.size();
            for (int i = 0; i < count; ++i) {
                stackDelete.push(stackAppend.pop());
            }
            return stackDelete.pop();
        }
    }
}
```
### 复杂度
* 时间复杂度:deleteHead 和 appendTail 平均时间复杂度 O(1)
* 空间复杂度: O(N)

## 优化 : 可以适用双端队列
```java

```

## 测试用例
["CQueue","appendTail","deleteHead","deleteHead","deleteHead"]  
[[],[3],[],[],[]]  

## leetcode链接
[leetcode链接题目链接](https://leetcode-cn.com/problems/yong-liang-ge-zhan-shi-xian-dui-lie-lcof/)  