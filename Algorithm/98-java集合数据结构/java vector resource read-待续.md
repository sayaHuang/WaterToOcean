[TOC]

# java vector resource read

> Vector类实现可增长的对象数组。像数组一样，它包含可以使用整数索引访问的组件。但是，Vector的大小可以根据需要增大或缩小，以适应创建Vector之后添加和删除项目的需要。
> 
> 每个Vector都尝试通过维护一个Capacity和一个CapacityIncrement来优化存储管理。容量始终至少与Vector大小一样大；它通常较大，因为将元素添加到Vector中时，向量的存储量会按容量增加的大小成块增加。应用程序可以在插入大量组件之前增加向量的容量；这减少了增量重新分配的数量。
> 
> 此类的 iterator 和 listIterator 方法返回的迭代器是快速失败的：如果在创建迭代器后的任何时间以任何方式对向量进行结构修改，除非通过迭代器自己的remove或add方法，否则迭代器将抛出ConcurrentModificationException。因此，面对并发修改，迭代器将迅速而干净地失败，而不是冒着在未来不确定的时间,犯着武断的，不确定的行为的风险。元素方法返回的枚举不是快速失败的。如果在创建枚举后的任何时间对Vector进行结构修改，则枚举的结果是不确定的。
> 
> 注意，迭代器的快速失败行为无法得到保证，因为通常来说，在存在不同步的并发修改的情况下，不可能做出任何严格的保证。快速失败的迭代器会尽最大努力抛出ConcurrentModificationException。因此，编写依赖于此异常的程序的正确性是错误的：迭代器的快速失败行为应仅用于检测错误。从Java 2平台v1.2开始，对该类进行了改进以实现List接口，使其成为Java Collections Framework的成员。与新的集合实现不同，Vector是同步的。如果不需要线程安全的实现，建议使用ArrayList代替Vector。
> 

Vecotor 中的方法参考网址 :
[Class Vector](https://docs.oracle.com/javase/9/docs/api/java/util/Vector.html)