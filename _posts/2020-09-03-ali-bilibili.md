---
layout: post
title: '阿里P7程序员 正确率只有50%的java基础题，你能挑战吗？'
subtitle: 'bilibili大法好'
date: 2020-09-03
author: Dave
cover: ''
tags: Java 
---



**最前面**

偶然看到的一个视频 bilibili整天都推送这些给我

- [阿里P7程序员 正确率只有50%的java基础题，你能挑战吗？](https://www.bilibili.com/video/BV1jv41167HC)


>测试环境：jdk13 idea2019.3

- 题目1

```java
float a=0.125f;
double b=0.125d;
System.out.println((a-b)==0);//true
```

- 题目2

```java
double c=0.8;
double d=0.7;
double e=0.6;
//c-d 与 d-e 相等吗
System.out.println(c-d);
System.out.println(d-e);
System.out.println(c-d==d-e);
/*
0.10000000000000009
0.09999999999999998
false
*/
```

- 题目3

```java
System.out.println(1.0/0);
/*
Infinity
*/
```

- 题目4

```java
System.out.println(0.0/0.0);
/*
NaN
*/
```

- 题目5

**>>** 与 **>>>**区别？

```java
/*
>>:右移运算符
高位的空位补符号位，即正数补零，负数补1。符号位不变。

1在32位二进制中表示为：11111111 11111111 11111111 11111111，-1>>1：按位右移，符号位不变，仍旧得到11111111 11111111 11111111 11111111，因此值仍为-1

>>>:二进制右移补零操作符（无符号右移）

左操作数的值按右操作数指定的位数右移，移动得到的空位以零填充，如value >>> num中，num指定要移位值value 移动的位数。*/

System.out.println(1.1f>>1);
System.out.println(1.1f>>>1);
/*
Error:(21, 32) java: 二元运算符 '>>>' 的操作数类型错误
第一个类型:  float
第二个类型: int
*/
```


- 题目6

某个类有两个重载方法：void f(String s)和void f(Integer s)，那么f(null)调用哪一个？


**答案： 编译出错**

>dave类中的dave方法 重载两个

```java

public dave(String cname) {
    System.out.println(cname);
}
public dave(Integer cname) {
    System.out.println(cname);
}

dave dave9=new dave(null);
/*
Error:(60, 20) java: 对dave的引用不明确
senstimeCoding.dave 中的构造器 dave(java.lang.String) 和 senstimeCoding.dave 中的构造器 dave(java.lang.Integer) 都匹配
*/

dave dave9=new dave((Integer) null);//输出null
dave dave9=new dave((String) null);//输出null
```

- 题目7

某个类有两个重载方法：void g(double d)和void g(Integer i)，那么g(1)调用哪一个？

**答案： g(double d)**

>dave类中的dave方法 重载两个

```java
public dave(double cname) {
    System.out.println(cname);
}
public dave(Integer cname) {
    System.out.println(cname);
}

dave dave9=new dave(1);//
/*
1.0
*/
```

- 题目8

String a=null;switch(a)匹配case中的哪一项？

>先说switch(),java 1.6(包括)以前，只是支持等价成int 基本类型的数据:byte,short,char,int。1.7加入的新特性可以支持String类型的数据。

**switch()的参数类型可以是：int，byte，short；String；char；enum**

**答案：抛出异常**



```java
String a = null;
switch(a){
    case "1":
        System.out.println("我是1");break;
    case "2":
        System.out.println("我是2");break;
    case "null":
        System.out.println("我是\"null\"");break;
    //case null:
    //    System.out.println("我是null");break; //这个写法编译出错
    default:
        System.out.println("我是default");break;
}
/*
Exception in thread "main" java.lang.NullPointerException
	at senstimeCoding.daveTest.main(daveTest.java:60)
*/
```
- 题目9

<String, T, Alibab> String get(String string,T t){ return string;} 此方法：

A 编译出错，第一个String处      
B 编译出错，T处
C 编译出错，Alibab处            
D 编译正确

**答案： D？**



```java
    <String, T, dave> String get(String string,T t){ return string;}
/*
居然不报错
*/
```

- 题目10

hashmap初始容量1000，即new HashMap(1000),当往里面put1000个元素时，需要resize几次？

备注：初始化那一次不算

**答案：**

这个要慢慢说 这道题我会

涉及的源码部分//jdk13

- put(K key, V value)

```java
public V put(K key, V value) {
    return putVal(hash(key), key, value, false, true);
}
```

- put(K key, V value)

```java
final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
                   boolean evict) {
        Node<K,V>[] tab; Node<K,V> p; int n, i;
        if ((tab = table) == null || (n = tab.length) == 0)
        //第一次调用put操作时，table都是null，所以必走这一步
            n = (tab = resize()).length;//resize()的作用就是开辟数组空间，此时tab就是新开辟数组的引用
        //开辟出新数组之后，计算当前key应该放在哪个slot中 
        //1.索引位置没有元素 直接插入数组上
        if ((p = tab[i = (n - 1) & hash]) == null)//i = (n - 1) & hash，计算索引
            tab[i] = newNode(hash, key, value, null);
        else {
            //第一次调用put的时候，下面代码都不会执行
            Node<K,V> e; K k;
            if (p.hash == hash &&
                ((k = p.key) == key || (key != null && key.equals(k))))
                e = p;//2.更新已有key的value
            else if (p instanceof TreeNode)//4.如果已经转树 就插入树上
                e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
            else {//3.链表的追加插入，尾插
                for (int binCount = 0; ; ++binCount) {
                    if ((e = p.next) == null) {
                        p.next = newNode(hash, key, value, null);//插入后 需要判断是否转树
                        if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                            treeifyBin(tab, hash);
                        break;
                    }
                    if (e.hash == hash &&
                        ((k = e.key) == key || (key != null && key.equals(k))))
                        break;
                    p = e;
                }
            }
            if (e != null) { // existing mapping for key
                V oldValue = e.value;
                if (!onlyIfAbsent || oldValue == null)
                    e.value = value;
                afterNodeAccess(e);
                return oldValue;
            }
        }
        ++modCount;
        //put完之后，size值会加1，如果此时大于阈值，将会扩容。
        if (++size > threshold)
            resize();
        afterNodeInsertion(evict);
        return null;
    }
```

- tableSizeFor(int cap)

```java
public static int tableSizeFor(int cap) {
    /*
    * @Param [cap] 
    * @description   //返回不小于cap的2的整数幂，比如tableSizeFor(2)返回2，tableSizeFor(3)返回4
    * @return int   
    **/
    int n = -1 >>> Integer.numberOfLeadingZeros(cap - 1);
    return (n < 0) ? 1 : (n >= MAXIMUM_CAPACITY) ? MAXIMUM_CAPACITY : n + 1;
}
```

- **resize()**

```java
final Node<K,V>[] resize() {
    	//原table数组赋值
        Node<K,V>[] oldTab = table;
    	//如果原数组为null，那么原数组长度为0
        int oldCap = (oldTab == null) ? 0 : oldTab.length;
    	//赋值阈值
        int oldThr = threshold;
    	//newCap 新数组长度
    	//newThr 下次扩容的阈值
        int newCap, newThr = 0;
    	// 1. 如果原数组长度大于0
        if (oldCap > 0) {
            //如果大于最大长度1 << 30 = 1073741824，那么阈值赋值为Integer.MAX_VALUE后直接返回
            if (oldCap >= MAXIMUM_CAPACITY) {
                threshold = Integer.MAX_VALUE;
                return oldTab;
            }
            // 2. 如果原数组长度的2倍小于最大长度，并且原数组长度大于默认长度16，那么新阈值为原阈值的2倍
            else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY &&
                     oldCap >= DEFAULT_INITIAL_CAPACITY)
                newThr = oldThr << 1; // double threshold
        }
    	// 3. 如果原数组长度等于0，但原阈值大于0，那么新的数组长度赋值为原阈值大小
        else if (oldThr > 0) // initial capacity was placed in threshold
            newCap = oldThr;
        else {               // zero initial threshold signifies using defaults
            // 4. 如果原数组长度为0，阈值为0，那么新数组长度，新阈值都初始化为默认值
            //这里主要是无参数构造的逻辑，newCap == 16, newThr == (int) 16 * 0.75
            newCap = DEFAULT_INITIAL_CAPACITY;
            newThr = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);
        }
    	// 5.如果新的阈值等于0
        if (newThr == 0) {
            //计算临时阈值
            float ft = (float)newCap * loadFactor;
            //新数组长度小于最大长度，临时阈值也小于最大长度，新阈值为临时阈值，否则是Integer.MAX_VALUE
            newThr = (newCap < MAXIMUM_CAPACITY && ft < (float)MAXIMUM_CAPACITY ?
                      (int)ft : Integer.MAX_VALUE);
        }
    	//计算出来的新阈值赋值给对象的阈值
        threshold = newThr;
        @SuppressWarnings({"rawtypes","unchecked"})
        //开辟空间在这里
    	//用新计算的数组长度新建一个Node数组，并赋值给对象的table
            Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap];
        table = newTab;
    	//后面是copy数组和链表数据逻辑
        if (oldTab != null) {
            //这里面的代码不会调用，因为oldTab == null。
            for (int j = 0; j < oldCap; ++j) {
                Node<K,V> e;
                if ((e = oldTab[j]) != null) {
                    oldTab[j] = null;
                    if (e.next == null)
                        newTab[e.hash & (newCap - 1)] = e;
                    else if (e instanceof TreeNode)
                        ((TreeNode<K,V>)e).split(this, newTab, j, oldCap);
                    else { // preserve order
                        Node<K,V> loHead = null, loTail = null;
                        Node<K,V> hiHead = null, hiTail = null;
                        Node<K,V> next;
                        do {
                            next = e.next;
                            if ((e.hash & oldCap) == 0) {
                                if (loTail == null)
                                    loHead = e;
                                else
                                    loTail.next = e;
                                loTail = e;
                            }
                            else {
                                if (hiTail == null)
                                    hiHead = e;
                                else
                                    hiTail.next = e;
                                hiTail = e;
                            }
                        } while ((e = next) != null);
                        if (loTail != null) {
                            loTail.next = null;
                            newTab[j] = loHead;
                        }
                        if (hiTail != null) {
                            hiTail.next = null;
                            newTab[j + oldCap] = hiHead;
                        }
                    }
                }
            }
        }
        //返回桶数组的引用
        return newTab;
    }
```




首先，三个构造方法都是懒加载，都没有第一时间开辟桶数组的空间；

初始容量 不一定等于初始化完成后底层数组实际的容量，因为存在阈值的计算；

也不是初始容量是多少开始就能存多少个元素，因为存在负载因子，在底层数组还没满的时候就会进行扩容；

**下表为不同构造方法下还未put时参数的初始值**

|**构造器**	|**this.loadFactor**    |**this. threshold**    |**this.table**    |**this.size**    |
|:---------|:---------:|:---------:|:---------:|:---------:|
|HashMap(int initialCapacity, float loadFactor)	|loadFactor    |tableSizeFor(initialCapacity)    |null    |0    |
|HashMap(int initialCapacity)	|DEFAULT_LOAD_FACTOR   |tableSizeFor(initialCapacity)    |null    |0    |
|HashMap()	|DEFAULT_LOAD_FACTOR   |0   |null    |0    |



测试 1000

```java
Map<Integer, Integer> map1 = new HashMap<>(1000);
for(int i=0;i<1000;i++){
    map.put(i, 1);
}
```



|**i**	|**数组长度** |**阈值threshold**    |**是否resize()**    |
|:---------|:---------:|:---------:|:---------:|
|-1(刚new的时候)	|0    |  1024 |   |
|0(第一次put的时候)	|1024    |  768 | 第一扩容resize()  |
|768(第769次put的时候)	|2048    |  1538 | 再次扩容resize()  |

所以1000的时候，不考虑第一扩容的情况下 会有一次resize()。

测试 10000

```java
Map<Integer, Integer> map1 = new HashMap<>(10000);
for(int i=0;i<10000;i++){
    map.put(i, 1);
}
```



|**i**	|**数组长度** |**阈值threshold**    |**是否resize()**    |
|:---------|:---------:|:---------:|:---------:|
|-1(刚new的时候)	|0    |  16384 |   |
|0(第一次put的时候)	|16384    |  12288 | 第一扩容resize()  |

put执行10000次也到不了阈值

所以10000的时候，不考虑第一扩容的情况下 不会再次resize()。


这里来考虑另一种 无参构造下的

```java
Map<Integer, Integer> map = new HashMap<>();
for(int i=0;i<1000;i++){
    map.put(i, 1);
}
```


|**i**	|**数组长度** |**阈值threshold**    |**是否resize()**    |
|:---------|:---------:|:---------:|:---------:|
|-1(刚new的时候)	|0    |  0 |   |
|0(第一次put的时候)	|16    |  12 | 第一扩容resize()  |
|24(第25次put的时候)	|32    |  24 | 再次扩容resize()  |
|48(第49次put的时候)	|64    |  48 | 再次扩容resize()  |
|......	|......    |  ...... | ......  |


所以阿里巴巴开发手册里面的指定map容量 是有必要的

- [由阿里巴巴Java开发规约HashMap条目引发的故事](https://zhuanlan.zhihu.com/p/30360734)

- [疫苗：JAVA HASHMAP的死循环](https://coolshell.cn/articles/9606.html?spm=5176.100239.blogcont225660.23.OtGRFx)
