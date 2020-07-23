---
layout: post
title: 'Java 中的栈'
subtitle: '关于Stack的几种实现'
date: 2020-05-27
author: Dave
tags: Java 
---

![](https://raw.githubusercontent.com/dendyikbc/PicGoBed/master/img/STACK.jpg)

## JAVA中堆栈的实现
- 不建议使用Stack类

```java
//不建议使用原始栈的写法，已经out了
//Stack是JDK 1.0的产物，继承自 Vector，复用了大量vector的方法，底层还是数组，官方已经不推荐使用了
Stack<Integer> stack = new Stack<>();
stack.push(127);//入栈
stack.pop();//出栈
```
### 利用JAVA现有类
- **ArrayDeque类**

```java
//简洁版本
Deque<Integer> stack = new ArrayDeque<>(32);
stack.addLast(127);//入栈
stack.removeLast();//出栈


//完整语义版本
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

- **LinkedList类**

```java
//简洁版本
Deque<Integer> stack = new LinkedList<>();
stack.addLast(127);//入栈
stack.removeLast();//出栈

//完整语义版本
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




### 实现MyStack类
- [Java在线工具](https://www.w3cschool.cn/tryrun/runcode?lang=java)
>这部分直接可以在线验证，在线编译器参照如上

1.2.3.来自于[Java 实现栈的三种方式](https://blog.csdn.net/weixin_43533825/article/details/96708590?utm_medium=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-2.compare&depth_1-utm_source=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-2.compare)，这里就直接搬运轮子了。


- **1.数组的方式实现**

```java
import java.util.Arrays;
/**
 * 数组实现栈
 * @param <T>
 */
class Mystack1<T> {
    //实现栈的数组
    private Object[] stack;
    //数组大小
    private int size;
 
    Mystack1() {
        stack = new Object[10];//初始容量为10
    }
 
    //判断是否为空
    public boolean isEmpty() {
        return size == 0;
    }
 
    //返回栈顶元素
    public T peek() {
        T t = null;
        if (size > 0)
            t = (T) stack[size - 1];
        return t;
    }
    
    public void push(T t) {
        expandCapacity(size + 1);
        stack[size] = t;
        size++;
    }
 
    //出栈
    public T pop() {
        T t = peek();
        if (size > 0) {
            stack[size - 1] = null;
            size--;
        }
        return t;
    }
 
    //扩大容量
    public void expandCapacity(int size) {
        int len = stack.length;
        if (size > len) {
            size = size * 3 / 2 + 1;//每次扩大50%
            stack = Arrays.copyOf(stack, size);
        }
    }
}
 
public class Main {
    public static void main(String[] args) {
        Mystack1<String> stack = new Mystack1<>();
        System.out.println(stack.peek());
        System.out.println(stack.isEmpty());
        stack.push("java");
        stack.push("is");
        stack.push("beautiful");
        stack.push("language");
        System.out.println(stack.pop());
        System.out.println(stack.isEmpty());
        System.out.println(stack.peek());
    }
}
/*
null
true
language
false
beautiful
*/
```
- **2.链表的方式实现**

```java
/**
 * 链表实现栈
 *
 * @param <T>
 */
class Mystack2<T> {
    //定义链表
    class Node<T> {
        private T t;
        private Node next;
    }
 
    private Node<T> head;
 
    //构造函数初始化头指针
    Mystack2() {
        head = null;
    }
 
    //入栈
    public void push(T t) {
        if (t == null) {
            throw new NullPointerException("参数不能为空");
        }
        if (head == null) {
            head = new Node<T>();
            head.t = t;
            head.next = null;
        } else {
            Node<T> temp = head;
            head = new Node<>();
            head.t = t;
            head.next = temp;
        }
    }
 
    //出栈
    public T pop() {
        T t = head.t;
        head = head.next;
        return t;
    }
 
    //栈顶元素
    public T peek() {
        T t = head.t;
        return t;
    }
 
    //栈空
    public boolean isEmpty() {
        if (head == null)
            return true;
        else
            return false;
    }
}
 
public class Main {
    public static void main(String[] args) {
        Mystack2 stack = new Mystack2();
        System.out.println(stack.isEmpty());
        stack.push("Java");
        stack.push("is");
        stack.push("beautiful");
        System.out.println(stack.peek());
        System.out.println(stack.peek());
        System.out.println(stack.pop());
        System.out.println(stack.pop());
        System.out.println(stack.isEmpty());
        System.out.println(stack.pop());
        System.out.println(stack.isEmpty());
    }
}

/*
true
beautiful
beautiful
beautiful
is
false
Java
true
*/

```
- **3.LinkedList的方式实现**

```java
import java.util.LinkedList;
 
/**
 * LinkedList实现栈
 *
 * @param <T>
 */
class ListStack<T> {
    private LinkedList<T> ll = new LinkedList<>();
 
    //入栈
    public void push(T t) {
        ll.addFirst(t);
    }
 
    //出栈
    public T pop() {
        return ll.removeFirst();
    }
 
    //栈顶元素
    public T peek() {
        T t = null;
        //直接取元素会报异常，需要先判断是否为空
        if (!ll.isEmpty())
            t = ll.getFirst();
        return t;
    }
 
    //栈空
    public boolean isEmpty() {
        return ll.isEmpty();
    }
}
 
public class Main {
    public static void main(String[] args) {
        ListStack<String> stack = new ListStack();
        System.out.println(stack.isEmpty());
        System.out.println(stack.peek());
        stack.push("java");
        stack.push("is");
        stack.push("beautiful");
        System.out.println(stack.peek());
        System.out.println(stack.pop());
        System.out.println(stack.isEmpty());
        System.out.println(stack.peek());
    }
}
/*
true
null
beautiful
beautiful
false
is
*/
```



dave的补充
- **4.双端队列的方式实现[推荐]**

```java
import java.util.Deque;
import java.util.ArrayDeque;
/**
 * 使用队列实现自定义堆栈
 * 1、弹
 * 2、压
 * 3、获取头
 * @author Administrator
 *
 * @param <E>
 */
class MyStack<E> {
       //容器
       private Deque<E> container=new ArrayDeque<E>();
       private int cap=9;
       
       public MyStack(int cap) {
              super();
              this.cap = cap;
       }

       //压栈
       public boolean push(E e){
           if(container.size()+1>cap){
                     return false;
              }
              return container.offerLast(e);
              
       }
       //弹栈
       public E pop(){
              return container.pollLast();
       }
       //获取
       public E peek(){
              return container.peekLast();
       }
       
       public int size(){
              return this.container.size();       
       }
}
  
  
  
  
//测试自定义堆栈
public class Main{
  
       /**
        * @param args
        */
       public static void main(String[] args) {
              MyStack<String> nameBack =new MyStack<String>(32);
              nameBack.push("dave");
              nameBack.push("dendy");
              nameBack.push("nora");
              nameBack.push("cathy");
              
              System.out.println("size:"+nameBack.size());
              
              //遍历
              String item=null;
              while(null!=(item=nameBack.pop())){
                     System.out.println(item);
              }
       }
}

/*
size:4
cathy
nora
dendy
dave

*/
```

- **5.单向队列的方式实现[需要两个单向队列]**

```java
/*
1.现有两个队列 q1 和 q2，入栈则将元素加到 q1

2.出栈的时候先判读 q1 是否为空，因为 q1 中的元素总是后进来的，后进先出，
除了队列的最后一个元素，将其它元素添加到 q2，q1 的最后一个元素出队

3.出栈的时候如果在 2 中判断 q1 为空，除了 q2 的最后一个元素，
将 q2 中其它元素添加到 q1，然后 q2 中的最后一个元素出队

*/
import java.util.Queue;
import java.util.LinkedList;
/**
 * 使用队列实现自定义堆栈
 * 1、弹
 * 2、压
 * 3、获取头
 * @author Administrator
 *
 * @param <E>
 */
class MyStack<E> {
       //容器
    private Queue<E> q1 = new LinkedList<E>();
    private Queue<E> q2 = new LinkedList<E>();

    public MyStack() {
    }

    public boolean push(E e) {
        return q1.offer(e);
    }

    public E pop() {
        if (q1.isEmpty() && q2.isEmpty()) {
            return null;
        }
        // 先判断 q1 是否为空 
        if (!q1.isEmpty()) {
            int size = q1.size();
            for (int i = 0; i < size - 1; i++) {
                q2.offer(q1.poll());
            }
            return q1.poll();
        } else {
            int size = q2.size();
            for (int i = 0; i < size - 1; i++) {
                q1.offer(q2.poll());
            }
            return q2.poll();
        }
    }

    public int size() {
        return q1.size() + q2.size();
    }
}
  
  
  
  
//测试自定义堆栈
public class Main{
  
       /**
        * @param args
        */
       public static void main(String[] args) {
              MyStack<String> nameBack =new MyStack<String>();
              nameBack.push("dave");
              nameBack.push("dendy");
              nameBack.push("nora");
              nameBack.push("cathy");
              
              System.out.println("size:"+nameBack.size());
              
              //遍历
              String item=null;
              while(null!=(item=nameBack.pop())){
                     System.out.println(item);
              }
       }
}
/*
size:4
cathy
nora
dendy
dave

*/
```


说到两个单向队列实现一个堆栈，就不得不提**一个队列的实现[需要两个栈]**

```java
 /*
1.有两个栈 stack1 和 stack2

2.入队列的时候只往 stack1 添加元素就行

3.出队列的时候先判断 stack2 是否为空，stack2 中的元素都是先进来的，先进先出。
如果 stack2 不为空，则直接弹出 stack2 的栈顶元素。如果为空，则将 stack1 的
元素添加到 stack2 中，然后弹出 stack2 的栈顶元素

 */
import java.util.Queue;
import java.util.Stack;
/**
 * 使用队列实现自定义堆栈
 * 1、弹
 * 2、压
 * 3、获取头
 * @author Administrator
 *
 * @param <E>
 */
class MyQueue<E> {
       //容器
    private Stack<E> stack1 = new Stack<E>();
    private Stack<E> stack2 = new Stack<E>();

    /**
     * 添加元素到队列
     *
     * @param num
     * @return
     */
    public E offer(E e) {
        return stack1.push(e);
    }

    /**
     * 队列弹出元素
     *
     * @return
     */
    public E poll() {
        if (stack1.isEmpty() && stack2.isEmpty()) {
            return null;
        }
        // 先判断 stack2
        if (!stack2.isEmpty()) {
            return stack2.pop();
        }
        // 将 stack1 的元素放入 stack2
        int size = stack1.size();
        for (int i = 0; i < size; i++) {
            stack2.push(stack1.pop());
        }
        return stack2.pop();
    }

    public int size() {
        return stack1.size() + stack2.size();
    }
}
  
  
  
  
//测试自定义队列
public class Main{
  
       /**
        * @param args
        */
       public static void main(String[] args) {
              MyQueue<String> nameBack =new  MyQueue<String>();
              nameBack.offer("dave");
              nameBack.offer("dendy");
              nameBack.offer("nora");
              nameBack.offer("cathy");
              
              System.out.println("size:"+nameBack.size());
              
              //遍历
              String item=null;
              while(null!=(item=nameBack.poll())){
                     System.out.println(item);
              }
       }
}
/*
size:4
dave
dendy
nora
cathy
*/

```




