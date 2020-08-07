---
layout: post
title: 'Deque接口的实现类'
subtitle: 'ArrayDeque与LinkedList实现Deque接口'
date: 2020-06-01
author: Dave
tags: Java 
---

![](https://raw.githubusercontent.com/dendyikbc/PicGoBed/master/img/deque.jpg)

参考资料：(源码对照比较好)
- [JAVA-QUEUE类图](https://blog.csdn.net/napo_leon/article/details/47830225)
>建议先点进去看类图，有个认知
- [Java™ Platform, Standard Edition 8 API Specification](https://docs.oracle.com/javase/8/docs/api/?java/util/LinkedList.html)
- [Java集合的小抄 ](https://www.runoob.com/w3cnote/java-collections.html)
## Deque接口的实现类
>Queue接口作为队列数据结构，java在实现的时候，直接定义了Deque接口（双端队列）来继承Queue接口，并且只实现Deque接口。所以我们在使用的时候直接使用的是Deque接口的实现类。
java中的Deque接口囊括了队列、双端队列、堆栈（Deque接口又定义了Stack的操作方法）这3种角色的功能。 


- ArrayDeque类实现Deque接口
- LinkedList类实现Deque接口
- ConcurrentLinkedDeque类实现Deque接口（本文暂时不讲）
>JDK1.7时，J.U.C包引入的一个集合工具类。ConcurrentLinkedDeque作为双端队列，可以当作“栈”来使用，并且高效地支持并发环境。ConcurrentLinkedDeque和ConcurrentLinkedQueue一样，采用了无锁算法，底层基于自旋+CAS的方式实现。


### ArrayDeque类实现Deque接口
![](https://raw.githubusercontent.com/dendyikbc/PicGoBed/master/img/ArrayDeque-Class-Pic.JPG)

**ArrayDeque类 特性**
- 用可变动态数组存储元素

```java
transient Object[] elements; // non-private to simplify nested class access
```
- 拥有head/tail这2个头尾指针；
**tail 是下一个等待插入元素的索引并非尾部元素的索引**

```java
//头部元素的索引
transient int head;
//尾部下一个将要被加入的元素的索引
transient int tail;
```

- 最小初始化容量为8，默认为16，是一个循环队列；


```java
//最小容量，必须为2的幂次方
private static final int MIN_INITIAL_CAPACITY = 8;

public ArrayDeque() {  
    elements = (E[]) new Object[16]; // 默认无参构造的数组长度大小 为16 
}  
  
public ArrayDeque(int numElements) {  
    allocateElements(numElements); // 需要的数组长度大小  
}  
  
public ArrayDeque(Collection<? extends E> c) {  
    allocateElements(c.size()); // 根据集合来分配数组大小  
    addAll(c); // 把集合中元素放到数组中  
}
```


- 扩容/初始化过程，遵循2的整数幂；

```java
//这地方初始化同hashmap源码，返回大于n的最小的2的整数幂
//值得一提的是，传入16返回32
//最大容量2^30
private void allocateElements(int numElements) {  
    int initialCapacity = MIN_INITIAL_CAPACITY;  
    // 找到大于需要长度的最小的2的幂整数。  
    // Tests "<=" because arrays aren't kept full.  
    if (numElements >= initialCapacity) {  
        initialCapacity = numElements;  
        initialCapacity |= (initialCapacity >>>  1);  
        initialCapacity |= (initialCapacity >>>  2);  
        initialCapacity |= (initialCapacity >>>  4);  
        initialCapacity |= (initialCapacity >>>  8);  
        initialCapacity |= (initialCapacity >>> 16);  
        initialCapacity++;  
  
        if (initialCapacity < 0)   // Too many elements, must back off  
            initialCapacity >>>= 1;// Good luck allocating 2 ^ 30 elements  
    }  
    elements = (E[]) new Object[initialCapacity];  
}

//扩容发生时，扩容为原来2倍
doubleCapacity(); 
//剩余源码 自己看
```
- 不能添加null值，不然会报空指针；

```java
public void addFirst(E e) {  
    if (e == null)  
        throw new NullPointerException();  
    // 本来可以简单地写成head-1，但如果head为0，减1就变为-1了，和elements.length - 1进行与操作就是为了处理这种情况，这时结果为elements.length - 1。  
    elements[head = (head - 1) & (elements.length - 1)] = e;  
    if (head == tail) // head和tail不可以重叠  
        doubleCapacity(); //双倍扩容 
}  
  
public void addLast(E e) {  
    if (e == null)  
        throw new NullPointerException();  
    // tail位置是空的，把元素放到这。  
    elements[tail] = e;  
    // 和head的操作类似，为了处理临界情况 (tail为length - 1时)，和length - 1进行与操作，结果为0。  
    if ( (tail = (tail + 1) & (elements.length - 1)) == head)  
        doubleCapacity();  
}  
  
public boolean offerFirst(E e) {  
    addFirst(e);  
    return true;  
}  
  
public boolean offerLast(E e) {  
    addLast(e);  
    return true;  
}
```
- 特别留意，内部通过二进制方式判断数组是否已满

```java
//public void addFirst(E e)
    if (head == tail) // head和tail不可以重叠  
        doubleCapacity(); 
//public void addLast(E e) 
    if ( (tail = (tail + 1) & (elements.length - 1)) == head)  
        doubleCapacity();  
```
- 内部虽然是数组，但不提供索引，随机访问并不友好；

```java
//删除指定元素源码 需要遍历整个队列
public boolean removeFirstOccurrence(Object o) {  
    if (o == null)  
        return false;  
    int mask = elements.length - 1;  
    int i = head;  
    E x;  
    while ( (x = elements[i]) != null) {  
        if (o.equals(x)) {  
            delete(i);  
            return true;  
        }  
        i = (i + 1) & mask; // 从头到尾遍历  
    }  
    return false;  
}  

```


**ArrayDeque基本方法**
```java
//ArrayDeque基本方法

//1.添加元素
addFirst(E e)//在数组前面添加元素
addLast(E e)//在数组后面添加元素
offerFirst(E e) //在数组前面添加元素，并返回是否添加成功
offerLast(E e) //在数组后天添加元素，并返回是否添加成功

//2.删除元素
removeFirst()//删除第一个元素，并返回删除元素的值,如果元素为null，将抛出异常
pollFirst()//删除第一个元素，并返回删除元素的值，如果元素为null，将返回null
removeLast()//删除最后一个元素，并返回删除元素的值，如果为null，将抛出异常
pollLast()//删除最后一个元素，并返回删除元素的值，如果为null，将返回null
removeFirstOccurrence(Object o) //删除第一次出现的指定元素
removeLastOccurrence(Object o) //删除最后一次出现的指定元素

//3.获取元素
getFirst() //获取第一个元素,如果没有将抛出异常
getLast() //获取最后一个元素，如果没有将抛出异常


//针对不同结构的语义方法内部实现也来自于以上基本方法
//栈的语义方法
public void push(E e) {  
    addFirst(e);  
}  
  
public E pop() {  
    return removeFirst();  
}

//同时，个人在写Leetcode时，也经常直接使用
Deque<Integer> stack = new ArrayDeque<>(32);
stack.addLast(127);//入栈
stack.removeLast();//出栈

//来替代原始栈类
//不建议使用原始栈的写法，已经out了
//Stack是JDK 1.0的产物，继承自 Vector，复用了大量vector的方法，底层还是数组，官方已经不推荐使用了
Stack<Integer> stack = new Stack<>();
stack.push(127);//入栈
stack.pop();//出栈

```


**针对不同结构具备不同语义化方法**
- **单向队列**

```java
Deque<String> queue = new ArrayDeque<>(32);

//进队列
queue.offer("dave");
queue.offer("dendy");
//出队列
queue.poll();


//语义通用部分
//取队首和队尾
queue.peekFirst();
queue.peekLast();
//包含与否
queue.cotains("dave");
//验空
queue.isEmpty();
//获取 容量
queue.size();
```
- **双端队列**

```java
Deque<String> deque = new ArrayDeque<>(32);

//队首队尾插入
deque.offerFirst("dave");
deque.offerLast("dendy");
//队首队尾删除
deque.pollFirst();
deque.pollLast();

//语义通用部分
//取队首和队尾
deque.peekFirst();
deque.peekLast();
//包含与否
deque.cotains("dave");
//验空
deque.isEmpty();
//获取 容量
deque.size();
```
- **堆栈**

```java
Deque<String> stack = new ArrayDeque<>(32);

//入栈
stack.push("dave");
stack.push("dendy");
//出栈
stack.pop();
//取栈顶
stack.peek();

//语义通用部分
//取栈顶和栈底
stack.peekFirst();
stack.peekLast();
//包含与否
stack.cotains("dave");
//验空
stack.isEmpty();
//获取 容量
stack.size();
```

### LinkedList类实现Deque接口
LinkedList实现了Deque和Queue接口，也可以实现队列、双端队列、堆栈（Deque接口又定义了Stack的操作方法）这3种角色的功能。
![](https://raw.githubusercontent.com/dendyikbc/PicGoBed/master/img/LinkedList-Class-Pic.JPG)
**LinkedList类 特性**
- 用双向链表存储元素，维护的是一个first和last指针，首尾插入/删除效率高，但同时存储元素时需要额外空间存储前驱与后继的引用

```java
private static class Node<E> {
        E item;
        Node<E> next;
        Node<E> prev;

        Node(Node<E> prev, E element, Node<E> next) {
            this.item = element;
            this.next = next;
            this.prev = prev;
        }
    }

```
- 允许元素为null
- fail-fast机制
- 非线程安全集合类
**LinkedList基本方法**

```java
//1.基本方法
add(E e) //添加一个元素
add(int index, E element)  //添加一个元素到指定位置
addFirist(Object o)
addLast(Object o)
get(int index) //获取指定位置的元素
getFirst()
getLast()
remove(Object o) //删除指定的元素
removeFirst()
removeLast()
set(int index, E element) //在指定位置节点设置值
indexOf(Object o) //查询指定元素值首次出现所在位置，如果无此节点则返回-1
lastIndexOf(Object o)//返回节指定元素最后出现的位置，如果无此节点则返回-1
contains(Object o)//判断链表节点对象中是否含有o

size()
isEmpty()

//2.针对队列/堆栈的语义化方法
//注意，LinkedList中add和offer的尽量不要混用，针对队列和堆栈时用offer

/*
offer属于 offer in interface Deque<E>;
add 属于 add in interface Collection<E>。   

当队列为空时候，add方法会报错，而offer方法会返回false。

作为List使用时,一般采用add/get方法来 压入/获取对象。

作为Queue使用时,才采用 offer/poll/take等方法作为链表对象时
*/
peek() //获取头节点的元素值，但不删除头节点
poll() //获取头节点的元素值，并删除头节点
pop() //获取头节点的元素值，并删除头节点，头节点为空则抛出异常

offer(E e)  //添加新元素到末尾
push(E e) //添加新元素到头节点

toArray()//转数组


```


**针对不同结构具备不同语义化方法**
- **单向队列**

```java
//两个队列实现一个栈，说的就是单向队列
Queue<String> queue = new LinkedList<>();
//进队列
queue.offer("dave");
queue.offer("dendy");
//出队列
queue.poll();

```
- **双端队列**

```java
Deque<String> deque = new LinkedList<>();

```
- **堆栈**

```java
Deque<String> stack = new LinkedList<>();

//入栈
stack.push("dave");
stack.push("dendy");
//出栈
stack.pop();
//取栈顶
stack.peek();



//包含与否
stack.cotains("dave");
//验空
stack.isEmpty();
//获取 容量
stack.size();

```
### LinkedList 与 ArrayDeque 在实现Deque接口时的选择

>按照 **是否需要按索引进行操作** 或 **经常插入/删除** 去选择**LinkedList** 
 
