[TOC]

# java stack resource read

> Java Collection框架提供了一个Stack类，用于建模和实现Stack数据结构。 该课程基于后进先出的基本原则。 除了基本的推入和弹出操作外，该类还提供了空，搜索和查看的三个功能。 也可以说该类是Vector的扩展，并将其视为具有之前提到的五个函数的堆栈。 该类也可以称为Vector的子类。

## 堆类中的方法:

1. `Object push(Object element)` : 将元素压入栈的顶部。
2. `Object pop()` : 删除并返回栈的顶部元素。 如果在调用栈为空时调用pop（），则会抛出“ EmptyStackException”异常。
3. `Object peek()` : 返回位于栈顶部的元素，但不删除它。
4. `boolean empty()` : 如果栈顶部没有任何内容，则返回true。 否则，返回false。
5. `int search(Object element)` : 这个方法判断元素是否存在于栈中, 如果元素找到了,返回元素所在的位置,如果元素没有找到 ,返回-1.

## 示例 代码

```java
Stack<Integer> stack = new Stack<>();
stack.push(1);
stack.push(2);
stack.push(3);
stack.push(4);
System.out.println(stack);
System.out.println(stack.search(4));
stack.pop();
stack.pop();
Integer topElement = stack.peek();
System.out.println(topElement);
System.out.println(" 3的位置 " + stack.search(3));
```

## stack类源码解读

```java
class Stack<E> extends Vector<E> {
		//1. Stack<Integer> stack = new Stack<>();
		//1.1 Creates an empty Stack.构造方法
    public Stack() {
    }
		
		//2. stack.push(1);
		//2.1Pushes an item onto the top of this stack
    public E push(E item) {
      addElement(item);//参看vector中2.2条
      return item;
    }
    
    //3. stack.pop();
    //3.1 移出栈中最上面的元素,并且将该元素返回
    public synchronized E pop() {
        E       obj;
        int     len = size();

        obj = peek();
        removeElementAt(len - 1);//查看vector的3.2

        return obj;
    }
    
    //4. Integer topElement = stack.peek();
    public synchronized E peek() {
        int     len = size();

        if (len == 0)
            throw new EmptyStackException();
        return elementAt(len - 1);//查看vector的4.1
    }
    //5.System.out.println(stack.search(4));
    //返回对象在此堆栈上的从1开始的位置。
        //如果对象{@code o}作为此堆栈中的一项出现，则此方法返回到出现的最上层堆栈顶部到最上层的距离； 堆栈中最顶层的项目被认为距离1。 
        //如果没有找到返回 -1
        //{@code equals}方法用于将{@code o}与该堆栈中的项目进行比较。
    public synchronized int search(Object o) {
        int i = lastIndexOf(o);//查看vector的5.1

        if (i >= 0) {
            return size() - i;
        }
        return -1;
    }          
    
    //返回vector中得元素数量
    public synchronized int size() {
        return elementCount;
    }
}

```

```java
public class Vector<E>
    extends AbstractList<E>
    implements List<E>, RandomAccess, Cloneable, java.io.Serializable
{
    /**
     *
     * @serial
     */
    //Vector分量存储在其中的数组缓冲区。 
    //Vector的容量是此数组缓冲区的长度，并且至少大到足以包含所有Vector的元素。
    //Vector中最后一个元素之后的任何数组元素均为null。
    protected Object[] elementData;

    //此{@code Vector}对象中有效组件的数量。
    //组件{@code elementData [0]}至{@code elementData [elementCount-1]}是Vector中的所有的实际项目。
    protected int elementCount;

    //当vector的容量变得大于其容量时，vector的容量自动增加的量。 如果capacityIncrement小于或等于零，则vector的容量每次需要增长时都会加倍。
    protected int capacityIncrement;

		//2.2vector类中的方法
    //将指定的元素添加到此Vector的末尾，将其大小增加一。 如果指定的元素的容量大于vector容量，则该vector的容量会增加。
    public synchronized void addElement(E obj) {
        modCount++;
        add(obj, elementData, elementCount);
    }
    
    //2.3 此辅助方法从add（E）中分离出来，以使方法的字节码大小保持在35以下（-XX：MaxInlineSize默认值），这有助于在C1编译循环中调用add（E）。
    private void add(E e, Object[] elementData, int s) {
        if (s == elementData.length)
            elementData = grow();
        elementData[s] = e;
        elementCount = s + 1;
    }
    
    //2.4 当容量不够的时候,会调用grow方法
    private Object[] grow() {
        return grow(elementCount + 1);
    }
    //2.5 Arrays为处理数组的方法集合
    private Object[] grow(int minCapacity) {
        return elementData = Arrays.copyOf(elementData,
                                           newCapacity(minCapacity));
    }
    //2.6
    private static final int MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8;
    private int newCapacity(int minCapacity) {
        // overflow-conscious code
        int oldCapacity = elementData.length;
        int newCapacity = oldCapacity + ((capacityIncrement > 0) ?
                                         capacityIncrement : oldCapacity);
				// minCapacity overflow(数据溢出)
				// newCapacity /  minCapacity 同时overflow
        if (newCapacity - minCapacity <= 0) {
            if (minCapacity < 0) //overflow
                throw new OutOfMemoryError();
            return minCapacity;
        }
        return (newCapacity - MAX_ARRAY_SIZE <= 0)
            ? newCapacity
            : hugeCapacity(minCapacity);
    }
    //2.7
    private static int hugeCapacity(int minCapacity) {
        if (minCapacity < 0) // overflow
            throw new OutOfMemoryError();
        return (minCapacity > MAX_ARRAY_SIZE) ?
            Integer.MAX_VALUE :
            MAX_ARRAY_SIZE;
    }
    
    //3.2
    //删除指定位置的元.
    //vector中的大于index的每一个元素,都需要向下移动一位
    //vector的大小减一
    //index的范围在0~size()-1
    public synchronized void removeElementAt(int index) {
        if (index >= elementCount) {
            throw new ArrayIndexOutOfBoundsException(index + " >= " +
                                                     elementCount);
        }
        else if (index < 0) {
            throw new ArrayIndexOutOfBoundsException(index);
        }
        int j = elementCount - index - 1;
        if (j > 0) {
            System.arraycopy(elementData, index + 1, elementData, index, j);
        }
        modCount++;
        elementCount--;
        elementData[elementCount] = null; /* to let gc do its work */
    } 
    
    //4.1
    //返回指定位置的元素
    //这个方法同List接口中的get()方法, 实现代码一模一样
    public synchronized E elementAt(int index) {
        if (index >= elementCount) {
            throw new ArrayIndexOutOfBoundsException(index + " >= " + elementCount);
        }

        return elementData(index);
    } 
    //4.2从数组中获取元素
    @SuppressWarnings("unchecked")
    E elementData(int index) {
        return (E) elementData[index];
    } 
    
    //5.1
    public synchronized int lastIndexOf(Object o) {
        return lastIndexOf(o, elementCount-1);
    } 
    
    //5.2 返回此向量中最后一次出现的指定元素的索引,从index开始倒着搜索,如果没有 找到返回-1
    public synchronized int lastIndexOf(Object o, int index) {
        if (index >= elementCount)
            throw new IndexOutOfBoundsException(index + " >= "+ elementCount);

        if (o == null) {
            for (int i = index; i >= 0; i--)
                if (elementData[i]==null)
                    return i;
        } else {
            for (int i = index; i >= 0; i--)
                if (o.equals(elementData[i]))//见object中的5.3
                    return i;
        }
        return -1;
    }    
}
```

```java
public class Object{
		//5.3
    public boolean equals(Object obj) {
        return (this == obj);
    }
}
```

## C1-compiled loop

## synchronized

## @serial





