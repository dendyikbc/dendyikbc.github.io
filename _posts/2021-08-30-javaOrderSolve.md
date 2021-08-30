---
layout: post
title: 'Java中关乎自定义顺序问题的解决'
subtitle: '自定义顺序'
date: 2021-08-30
author: Dave
cover: ''
tags: Java
---

## Java 中关乎自定义顺序问题的解决
>小记一下，写个笔记。

**最前面**

问题抛出：Java中本身带顺序的数据结构有哪些？


强顺序

>默认为按key值 String字典序 int自然增序

- TreeMap
- TreeSet


默认插入顺序

- PriorityQueue
  >特别地，队顶元素为排序首位；存储元素大致为插入二叉树的顺序；poll输出时满足自定义顺序。
- List
- Arrays
- stream流
  
>以上均实现Comparator类，均可自定义顺序

## Comparator实现自定义顺序

>实现方式有两种
>第一种：定义一个新类去实现Comparator接口，重写其中的compare方法。
>第二种：new这个接口并重写里面的compare方法。

### 第一种主要为自定义类的排序

- 示例1:定义新类时继承Comparable接口，并重写
```java
//新类定义时，继承Comparable接口，并重写
import java.util.*;

public class ComparatorDemo {
    public static void main(String[] args) {
        List<Person> people = Arrays.asList(
                new Person("dave", 25),
                new Person("piccolo", 20),
                new Person("mia", 24)
        );
        Collections.sort(people);//这里会自动调用Person中重写的compareTo方法。
        System.out.println(people);
        //[{name=dave, age=25}, {name=mia, age=24}, {name=piccolo, age=20}]
    }
}

class Person implements Comparable<Person>{
    String name;
    int age;
    Person(String n, int a) {
        name = n;
        age = a;
    }
    @Override
    public int compareTo(Person person) {
        return name.compareTo(person.name);
        //return this.name - person.name;
    }
    @Override
    public String toString() {
        return String.format("{name=%s, age=%d}", name, age);
    }
}
```

- 示例2:已有类且不便于修改，则额外定义一个新类，继承Comparable接口，并重写
  
```java
//若已有类且不便于修改，则额外定义一个新类，继承Comparable接口，并重写
import java.util.*;

public class ComparatorDemo {
    public static void main(String[] args) {
        List<Person> people = Arrays.asList(
                new Person("dave", 25),
                new Person("piccolo", 20),
                new Person("mia", 24)
        );
        Collections.sort(people, new comparatorByName());
        System.out.println(people);
        //[{name=dave, age=25}, {name=mia, age=24}, {name=piccolo, age=20}]
    }
}

class comparatorByName implements Comparator<Person> {
    @Override
    public int compare(Person a, Person b) {
        return a.name.compareToIgnoreCase(b.name);
    }
}

class Person {
    String name;
    int age;
    Person(String n, int a) {
        name = n;
        age = a;
    }
    @Override
    public String toString() {
        return String.format("{name=%s, age=%d}", name, age);
    }
}
```

### 第二种最常用

>重写Comparator，不可以用基本类型。

- 1.实例Comparator 
- 2.匿名内部类实现
- 3.Lambda简化代码  

```java

String[] strArr ={"a","ddd","cda","bb","ba"};

//1.实例Comparator
Comparator<String> comparatorByDictOrder = new Comparator<String>() {
    @Override
    public int compare(String a, String b) {
        int lenDif = a.length() - b.length();
        return lenDif == 0 ? a.compareTo(b) : lenDif;
    }
};
Arrays.sort(strArr, comparatorByDictOrder);
printArr(strArr);
//2.匿名内部类实现Comparator
Arrays.sort(strArr, new Comparator<String>() {
    @Override
    public int compare(String a, String b) {
        int lenDif = a.length() - b.length();
        return lenDif == 0 ? a.compareTo(b) : lenDif;
    }
});
printArr(strArr);

//Lambda简化
Comparator<String> comparatorByDictOrderL = (String a, String b) ->
        ((a.length() - b.length()) == 0 ? a.compareTo(b) : (a.length() - b.length()));
Arrays.sort(strArr, comparatorByDictOrderL);
printArr(strArr);
/*printArr输出结果均为
当前数组中元素为：
a
ba
bb
cda
ddd
--------------------
*/

//printArr 为泛型打印数组元素
public static <E> void printArr(E[] arr) {
    System.out.println("当前数组中元素为：");
    Arrays.stream(arr).forEach(System.out::println);
    System.out.println("--------------------");
}
```

### 常用Comparator

**下文涉及Comparator，以Lambda简化为主。**


```java
//常用Comparator

//int型自定义Comparator实现
//1.数值增序
Comparator<Integer> comparatorByValue=(Integer a,Integer b) -> Integer.valueOf(a)-Integer.valueOf(b);
Comparator<Integer> comparatorByValue2=(Integer a,Integer b) -> Integer.compare(a,b);

//String型自定义Comparator实现
//1.字符串长度增序
Comparator<String> comparatorByStrLen=(String a, String b) -> a.length() - b.length();
//2.字符串大小增序
Comparator<String> comparatorByStrValue=(String a, String b) -> a.compareTo(b);
//3.按照字符串最后一个字符大小增序
Comparator<String> comparatorByLastCharValue = (String a, String b) ->
        (a.charAt(a.length() - 1) - b.charAt(b.length() - 1));
//4.字典序
Comparator<String> comparatorByDictOrderL = (String a, String b) ->
        ((a.length() - b.length()) == 0 ? a.compareTo(b) : (a.length() - b.length()));
```

## TreeMap

```java
import java.util.*;

public class test {
    public static void main(String[] args) {
        System.out.println("test");
        TreeMap<Integer, Integer> treeMap = new TreeMap<>();
        treeMap.put(0, 1);
        treeMap.put(1, 3);
        treeMap.put(2, 6);
        treeMap.put(5, 21);
        treeMap.put(3, 10);
        treeMap.put(4, 15);
        System.out.println("treeMap原始");//整型 自然按key增序列
        printMapByEntrySetL(treeMap);
        // 此时treeMap
        // key:  0  1  2   3   4   5
        // value:1  3  6  10  15  21
        TreeMap<String, String> treeMapS = new TreeMap<>();
        treeMapS.put("a", "dave");
        treeMapS.put("bb", "mia");
        treeMapS.put("cda", "invoker");
        treeMapS.put("ddd", "bella");
        treeMapS.put("ba", "jack");
        System.out.println("treeMapS原始");
        printMapByEntrySetL(treeMapS);//字符型 自然按key字典序（字典序下，同时满足字符长度顺序；同等长度比较字符大小）
    }// main end

    public static <E> void printMapByEntrySet(Map<E, E> map) {
        //定义泛型化的Map遍历输出方法----基于键值对Set
        // A:获取所有的键值对对象的集合
        Set<Map.Entry<E, E>> set = map.entrySet();
        System.out.println("当前Map中元素为：");
        System.out.println("key---value");
        // B:遍历键值对对象的集合，得到每一个键值对的对象
        for (Map.Entry<E, E> me : set) {
            // C:获取键和值
            E key = me.getKey();
            E value = me.getValue();
            System.out.println(key + "  ---  " + value);
        }
        System.out.println("--------------------");
    }

    //引入Lambda + Stream 改写
    public static <E> void printMapByEntrySetL(Map<E, E> map) {
        //定义泛型化的Map遍历输出方法----基于键值对Set
        // A:获取所有的键值对对象的集合
        Set<Map.Entry<E, E>> set = map.entrySet();
        System.out.println("当前Map中元素为：");
        System.out.println("key---value");
        // B:遍历键值对对象的集合，得到每一个键值对的对象
        set.stream().forEach(Entry -> System.out.println(Entry.getKey() + "  ---  " + Entry.getValue()));
        System.out.println("--------------------");
    }

    public static <E> void printMapByKeySet(Map<E, E> map) {
        //定义泛型化的Map遍历输出方法----基于键Set 查询值
        // A:获取所有的键
        Set<E> set = map.keySet();
        System.out.println("当前Map中元素为：");
        System.out.println("key---value");
        // B:遍历键的集合，获取得到每一个键
        for (E key : set) {
            // C:根据键查询值
            E value = map.get(key);
            System.out.println(key + "  ---  " + value);
        }
        System.out.println("--------------------");
    }

    //引入Lambda + Stream 改写
    public static <E> void printMapByKeySetL(Map<E, E> map) {
        //定义泛型化的Map遍历输出方法----基于键Set 查询值
        // A:获取所有的键
        Set<E> set = map.keySet();
        System.out.println("当前Map中元素为：");
        System.out.println("key---value");
        // B:键的集合stream流化并遍历查询并输出
        set.stream().forEach(key -> System.out.println(key + "  ---  " + map.get(key)));
        System.out.println("--------------------");
    }

}//test Class end
```

## TreeSet

>默认字典序

```java
TreeSet<String> treeSetS = new TreeSet<>();
treeSetS.add("a");
treeSetS.add("bb");
treeSetS.add("cda");
treeSetS.add("ddd");
treeSetS.add("ba");

System.out.println("treeSetS原始");
treeSetS.stream().forEach(System.out::println);
```

## PriorityQueue

- [PriorityQueue介绍](https://github.com/CarpenterLee/JCFInternals/blob/master/markdown/8-PriorityQueue.md)
  
特别注意，PriorityQueue底层由二叉小顶堆实现，默认为String字典序，Integer 自然增序。`内部排序结构保证堆顶元素（也为队顶元素peek）为排序的首位元素，其余元素内部排序不一定完全按照设计的顺序排列，仅在挨个poll时，完全符合自定义顺序。`

```java
Queue<String> pQueue=new PriorityQueue<String>();//comparatorByLastCharValue
pQueue.offer("bb");
pQueue.offer("a");
pQueue.offer("ba");
pQueue.offer("ddd");
pQueue.offer("cda");
// 直接输出
System.out.println("此时，队列存储顺序为：其实并不符合字典序,可以对应上二叉树的存储顺序。");
pQueue.stream().forEach(System.out::println);
System.out.println("-----------");
//stream().sorted()输出顺序为：
System.out.println("stream().sorted()输出顺序为：");
pQueue.stream().sorted(comparatorByLastCharValue).forEach(System.out::println);
System.out.println("-----------");
//依次poll
System.out.println("依次poll顺序为：");
while(!pQueue.isEmpty()){
    System.out.println(pQueue.poll());
}
System.out.println("-----------");
/*
此时，队列存储顺序为：其实并不符合字典序,可以对应上二叉树的存储顺序。
a
bb
ba
ddd
cda
-----------
stream().sorted()输出顺序为：
a
ba
cda
bb
ddd
-----------
依次poll顺序为：
a
ba
bb
cda
ddd
-----------
*/
```

## List & Arrays

```java
List<String> list = new ArrayList<>();
list.add("a");
list.add("bb");
list.add("cda");
list.add("ddd");
list.add("ba");
String[] strArr = list.toArray(new String[0]);//new String[0] 创建一个长度为 0 的数组

Arrays.sort(strArr,comparatorByDictOrderL);
printArr(strArr);

Collections.sort(list,comparatorByDictOrderL);
list.stream().forEach(System.out::println);

//printArr 为泛型打印数组元素
public static <E> void printArr(E[] arr) {
    System.out.println("当前数组中元素为：");
    Arrays.stream(arr).forEach(System.out::println);
    System.out.println("--------------------");
}
```

## stream

>stream 流很强大

```java
List<String> list = new ArrayList<>();
list.add("a");
list.add("bb");
list.add("cda");
list.add("ddd");
list.add("ba");
String[] strArr = list.toArray(new String[0]);//new String[0] 创建一个长度为 0 的数组

list.stream()
        .sorted(comparatorByDictOrderL)
        .forEach(System.out::println);
Arrays.stream(strArr)
        .sorted(comparatorByDictOrderL)
        .forEach(System.out::println);
//优先队列 在流的时候排序 比较容易实现
PriorityQueue<String> pQueue=new PriorityQueue<String>();
pQueue.offer("ba");
pQueue.offer("a");
pQueue.offer("bb");
pQueue.offer("ddd");
pQueue.offer("cda");
System.out.println("-----------");
pQueue.stream().sorted(comparatorByDictOrderL).forEach(System.out::println);
System.out.println("-----------");
PriorityQueue<Integer> queueMin = new PriorityQueue<Integer>();
queueMin.offer(3);
queueMin.offer(6);
queueMin.offer(2);
queueMin.offer(22);
queueMin.offer(13);
System.out.println("-----------");
queueMin.stream().sorted(comparatorByValue).forEach(System.out::println);
System.out.println("-----------");
```



