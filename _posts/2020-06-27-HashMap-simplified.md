---
layout: post
title: 'HashMap 总结速览版'
subtitle: '快速回顾HashMap'
date: 2020-06-27
author: Dave
tags: Java 
---

![](https://raw.githubusercontent.com/dendyikbc/PicGoBed/master/img/hashmap-quick-review.jpg)
##  HashMap 总结速览 快速回顾
>这部分速览，主要是方便快速复习

| Tables|        | 有序           | 允许元素重复  |
| ------|------- |:-------------:| -----:|
| Collection |      | 否 | 是|
| List |      | 是      |   是 |
| Set |AbstractSet | 否      |    否 |
|  |HashSet | 否      |    否 |
|  |TreeSet | 是（用二叉树排序）      |    否 |
|  Map|AbstractMap | 否      |  使用key-value来映射和存储数据，Key必须惟一，value可以重复   |
|  |HashMap | 否      |   同上  |
|  |TreeMap | 是（用二叉树排序）      |    同上 |

## HashMap特点
- **key-value 映射，key唯一，value可重复；**

    这部分基于hashing，任意长度输入 转换 成 固定长度 输出
    key → 对键调用hashCode()方法f(key)获取存储地址0x9527 → 在0x9527存储(key,value)
    ```java
        static final int hash(Object key) {
        int h;
        return key == null ? 0 : (h = key.hashCode()) ^ h >>> 16;//高16位^低16位 
        }
    ```

    值得注意的是，**Node对象的hash方法的值并非key的hashcode值，而是hashcode值经过二次加工得到（高低位异或，用于优化hash算法）**

    解释：与哈希寻址算法有关，哈希寻址采用hash & (table.length-1)，hash与高位为0 低位全1的数 按位与后得到的值>=0且<=这个二进制数，也就是说实际有效位数只有低位，一般在16位以内，这样key.hashCode 高16位就被浪费掉了，没有起到作用
    >HashMap 用 hash & (table.length-1)代替了 模运算 , 如果直接使用 hashCode() 的返回值的话, 只有hash code的低位(如果 table.length 是2的n次方, 只有最低的 n - 1 位)会参加运算, 高位即使发生变化也会产生碰撞. 而 hash() 方法把 hashCode 的高位与低位异或, 相当于高位也参加了运算, 能够减少碰撞，以此优化hash算法.


- **底层结构**

    | JDK版本|    底层结构    | 
    | ------|------- |
    | JDK1.8以前 |   数组+链表   | 
    | JDK1.8及以后 |   数组+链表+红黑树   |

    链表转红黑树 临界条件
    - slot内部链表长度>8 数组长度>64 转红黑树
    - slot内部链表长度>8 数组长度<64 进行数组扩容resize
        >链表长度遵循泊松分布，长度为8时 概率很小

    红黑树转链表 临界条件
    - 链表长度<6

- **允许null元素**
    >具体体现在：null可以作为一个键，但这样的键只有一个；可以有一个或多个键所对应的值为null。
    >
    >值得注意的是：当get()方法返回null值时，即可以表示 HashMap中没有该键，也可以表示该键所对应的值为null。因此，在HashMap中不能由get()方法来判断HashMap中是否存在某个键，而应该用containsKey()方法来判断。
- **负载因子默认为0.75，数组长度默认16，map填满0.75*size时map进行resize为原map两倍(位移运算)**

    ```java
    static final float DEFAULT_LOAD_FACTOR = 0.75F;
    static final int DEFAULT_INITIAL_CAPACITY = 16;
    //HashMap 构造函数里面 指定了最大初始化容量
    if (initialCapacity > 1073741824) {
                    initialCapacity = 1073741824;
                }
    ```
- **懒加载机制，第一次put数据的时候才创建**
- **缺省状态下并非线程安全**
- **map大小必须为2^n**

    ```JAVA
    new HashMap<String, String>(X);//返回>X的2的整数幂
    new HashMap<String, String>(1025);//实际为2048

    //源码中的cap
    static final int tableSizeFor(int cap) {
        int n = -1 >>> Integer.numberOfLeadingZeros(cap - 1);
        //numberOfLeadingZeros返回无符号整型i的最高非零位前面的0的个数，包括符号位在内
        // 0101 返回1 负数 返回0，符号位为1
        //-1。先取1的原码：00000000 00000000 00000000 00000001，得反码： 11111111 11111111 11111111 11111110，最后得补码： 11111111 11111111 11111111 11111111
        return n < 0 ? 1 : (n >= 1073741824 ? 1073741824 : n + 1);
    }

    ```
    为什么这么设定？
    - **优化取模运算性能**

        与运算符替代传统取余，要求2的整数幂
        ```JAVA
        final V putVal(int hash, K key, V value, boolean onlyIfAbsent, boolean evict) {
            HashMap.Node[] tab;
            int n;
            if ((tab = this.table) == null || (n = tab.length) == 0) {
                n = (tab = this.resize()).length;
            }

            Object p;
            int i;
            if ((p = tab[i = n - 1 & hash]) == null) {
                tab[i] = this.newNode(hash, key, value, (HashMap.Node)null);
            } else {
                ..........
            }
            ..............
        ```
    - **降低哈希冲突概率**

        2^n-1 为全1 hash&(table.length-1) 时 对hash更敏感，更容易均匀散列，从而降低冲突的发生




## HashMap的put写数据具体流程
put(key,value)

```java
    public V put(K key, V value) {
        return putVal(hash(key), key, value, false, true);
    }
    
    /**
     * Implements Map.put and related methods.
     *
     * @param hash hash for key
     * @param key the key
     * @param value the value to put
     * @param onlyIfAbsent if true, don't change existing value
     * @param evict if false, the table is in creation mode.
     * @return previous value, or null if none
     */
    final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
                   boolean evict) {
                   ......
                   
                   }
```
    
说到这里 更新两个有意思的方法 很方便

```java
    map.put(key, map.getOrDefault(key, 0) + 1);
    //getOrDefault(key, 0)有key就用key对应的value,没有就使用defaultValue，这里的defaultValue传入值为0
    //以前没怎么注意到hashmap这个方法 其实挺好用的 

    //当Map集合中有这个key时，就使用这个key值去获取相应value来使用，如果没有就使用默认值defaultValue
    default V getOrDefault(Object key, V defaultValue)
    //果传入key对应的value已经存在，就返回存在的value，不进行替换。如果不存在，就添加key和value，返回null
    V putIfAbsent(K key, V value)
```
**寻址算法**

**1.key 与 hashCode 高低位异或 获取hash字段**
```java
static final int hash(Object key) {
        int h;
        return key == null ? 0 : (h = key.hashCode()) ^ h >>> 16;//高16位^低16位 
    }
```
**2.hash&(table.length-1)得到slot 下标**
与高位为0 低位全1的数 按位与

**数据插入**

**3.根据slot槽内状况，状况不同，情况不同**

>源码里面斜的很清楚 顺下来就对了

- slot=null
>直接插入key-value
- slot!=null 引用的node还没有链化
>如果slot位置有元素，且满足key相等条件，替换旧值。如果key不相等，采用尾插法在slot->node后面追加一个node，形成链表
- slot!=null 引用的node已经链化
>遍历链表，如果有key满足相等条件，替换旧值；否则插入key-value到链表最后；插入之后如果当前链表长度大于TREEIFY_THRESHOLD，转换成红黑树结构。
- slot!=null 引用的node为TreeNode
>执行树的插入操作

## HashMap的resize扩容机制
双倍扩容
具体怎么移动元素的呢？

## 哈希冲突
>理论上不可避免，只能尽量避免

### hash函数要求
- 效率高
    >长文本也要高效计算出hash值
- hash值不能逆推原文
- 两次输入 只要有一点不同 hash值也应不一样
- 计算的hashcode尽可能要分散，也是为了尽可能降低哈希冲突

### JDK为什么引入红黑树
- 链表平均查找长度N/2
- 红黑树平均查找长度log(N)

| |    链表平均查找长度N/2    | 红黑树平均查找长度log(N)           | 说明  |
| ------|:-------:|:-------------:| -----:|
| N=8 |  4  | 3 | 有必要转红黑树|
| N=6 |  3  | 2.6      |   树的生成需要时间，性能差异不大时，链表结构更优 |

红黑树 特殊的二叉排序树 也可以有很多  
比如
红黑树的插入
红黑树的扩容



## 基本使用
```java
/*
* Map集合和Collection集合的区别？
* Map集合存储元素是成对出现的，Map集合的键是唯一的，就是可重复的。可以把这个理解为：夫妻对
* Collection集合存储元素是单独出现的，Collection的儿子Set是唯一的，List是可重复的，可以把这个理解为：光棍
* 
* 注意：
* Map集合的数据结构值针对键有效，限值无效
* Collection集合的数据结构是针对元素有效
* 
* */

//Map集合的功能概述：
//1：添加功能
V put(K key,V value);//添加元素
//如果键是第一次存储，就直接存储元素，返回null
//如果键不是第一次存储，就用值把以前的值替换掉，返回以前的值

//2：删除功能
void clear();//移除所有的键值对元素
V remove(Object key);//根据键删除键值对元素，并把值返回

//3：判断功能
boolean containsKey(Object key);//判断集合是否包含指定的键
boolean containsValue(Object value);//判断集合是否包含指定的值
boolean isEmpty();//判断集合是否为空

//4：获取功能
set<Map,Entry<E,V>> entrySet();获取键值对的对象集合
V get(Object key);//根据键获取值
Set<K> keySet();//获取集合中所有键的集合
Collection<V> values();//获取集合中所有值的集合

//5：长度功能
int size();//返回集合中的键值对的对数
```
**Map集合的遍历**

- 方式1，根据键查询值

```java
import java.util.HashMap;
import java.util.Map;
import java.util.Set;
 
/*
 * Map集合的遍历，根据键查询值
 *
 * 思路：
 * A:获取所有的键
 * B:遍历键的集合，获取得到每一个键
 * C:根据键查询值
 * */
 
public class IntegerDemo {
  public static void main(String[] args) {
    // TODO Auto-generated method stub
 
    Map<String, String> map = new HashMap<String, String>();
 
    map.put("mia", "Ma");
    map.put("dave", "Chou");
    map.put("nora", "Chen");
 
    System.out.println(map);
 
    // A:获取所有的键
    Set<String> set = map.keySet();
 
    // B:遍历键的集合，获取得到每一个键
    for (String key : set) {
      // C:根据键查询值
      String value = map.get(key);
      System.out.println(key + "---" + value);
    }
  }
}
```
- 方式2，根据键值对的对象查询键和值

```java
import java.util.HashMap;
import java.util.Map;
import java.util.Set;
 
/*
 * Map集合的遍历，根据对象查询键和值
 *
 * 思路：
 * A:获取所有的键值对对象的集合
 * B:遍历键值对对象的集合，得到每一个键值对的对象
 * C:获取键和值
 * */
 
public class IntegerDemo {
  public static void main(String[] args) {
    // TODO Auto-generated method stub
 
    Map<String, String> map = new HashMap<String, String>();
 
    map.put("mia", "Ma");
    map.put("dave", "Chou");
    map.put("nora", "Chen");
 
    System.out.println(map);
 
    // A:获取所有的键值对对象的集合
    Set<Map.Entry<String, String>> set = map.entrySet();
 
    // B:遍历键值对对象的集合，得到每一个键值对的对象
    for (Map.Entry<String, String> me : set) {
      // C:获取键和值
      String key = me.getKey();
      String value = me.getValue();
      System.out.println(key + "---" + value);
    }
  }
}
```

```java

/*
* 1:HashMap和Hashtable的区别？
* HashMap线程不安全，效率高，允许null键和null值
* Hashtable线程安全，效率低，不允许null键和null值
* 
* 2:List，Set，Map等接口是否都继承于Map接口？
* List，Set不是继承自Map接口，它们继承自Collection接口
* Map接口本身就是一个顶层接口
* */
import java.util.HashMap;
import java.util.Hashtable;
 
public class IntegerDemo {
  public static void main(String[] args) {
    // TODO Auto-generated method stub
 
    HashMap<String, String> hm = new HashMap<String, String>();
    Hashtable<String, String> ht = new Hashtable<String, String>();
 
    hm.put("hello", "world");
    hm.put("java", "c++");
    hm.put(null, "sql");
 
    ht.put("hello", "world");
    ht.put("java", "c++");
    ht.put(null, "sql");// Exception in thread "main"
              // java.lang.NullPointerException
  }
}


```

**HashMap与LinkedHashMap**
有轮子看了

附上一些HashMap博客
- [HashMap实现原理及源码分析](https://www.cnblogs.com/chengxiao/p/6059914.html#t3)
- [Java 8系列之重新认识HashMap](https://tech.meituan.com/2016/06/24/java-hashmap.html)
- [Java集合之LinkedHashMap](https://www.cnblogs.com/xiaoxi/p/6170590.html)
