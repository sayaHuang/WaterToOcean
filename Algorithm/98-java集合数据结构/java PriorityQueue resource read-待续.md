[TOC]

#  java PriorityQueue resource read

> 当元素需要 按照优先级来处理的时候可以使用PriorityQueue. 众所周知, queue时按照 `先进先出` 的算法处理元素的, 但是有时候queue中的元素需要按照优先级来处理. 所以诞生了PriorityQueue. PriorityQueue是建立在priority heap基础上的. 
> PriorityQueue中的元素默认按照自然排序, 或者按照queue在初始化的时候,在参数Comparator实例的接口中所给定的指定排序方式

## 使用注意点:

1. PriorityQueue不能添加null, 毕竟null无法比较大小
2. PriorityQueue的元素需要支持比较
3. PriorityQueue时没有边界的queue
4. 就指定的顺序而言，此队列的头是最小的元素。 如果多个元素的价值最小，那么头就是那些元素之一 - 关系被任意打破。
5. 它继承了AbstractQueue，AbstractCollection，Collection和Object类的方法。

## 使用代码:

```java
public static void main(String args[]) 
    { 
        // Creating empty priority queue 
        PriorityQueue<String> pQueue = new PriorityQueue<String>(); 
  
        // Adding items to the pQueue using add() 
        pQueue.add("C"); 
        pQueue.add("C++"); 
        pQueue.add("Java"); 
        pQueue.add("Python"); 
  
        // Printing the most priority element 
        System.out.println("Head value using peek function:"
                           + pQueue.peek()); 
  
        // Printing all elements 
        System.out.println("The queue elements:"); 
        Iterator itr = pQueue.iterator(); 
        while (itr.hasNext()) 
            System.out.println(itr.next()); 
  
        // Removing the top priority element (or head) and 
        // printing the modified pQueue using poll() 
        pQueue.poll(); 
        System.out.println("After removing an element"
                           + "with poll function:"); 
        Iterator<String> itr2 = pQueue.iterator(); 
        while (itr2.hasNext()) 
            System.out.println(itr2.next()); 
  
        // Removing Java using remove() 
        pQueue.remove("Java"); 
        System.out.println("after removing Java with"
                           + " remove function:"); 
        Iterator<String> itr3 = pQueue.iterator(); 
        while (itr3.hasNext()) 
            System.out.println(itr3.next()); 
  
        // Check if an element is present using contains() 
        boolean b = pQueue.contains("C"); 
        System.out.println("Priority queue contains C "
                           + "or not?: " + b); 
  
        // Getting objects from the queue using toArray() 
        // in an array and print the array 
        Object[] arr = pQueue.toArray(); 
        System.out.println("Value in array: "); 
        for (int i = 0; i < arr.length; i++) 
            System.out.println("Value: " + arr[i].toString()); 
    } 
```

## 源码分析
```java

```