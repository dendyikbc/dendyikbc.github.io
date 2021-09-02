---
layout: post
title: 'Java中List 与 Arrays 相互转换'
subtitle: 'List <--> Arrays '
date: 2021-09-02
author: Dave
cover: ''
tags: Java
---

## Java中List 与 Arrays 相互转换

>想起了，就补充一下。

这里排除了最简单的for循环构造此类方法。


```java
//下面将用到
List<Integer> list=new ArrayList<>();
list.add(1);list.add(2);list.add(3);list.add(4);list.add(5);
int[] arr=new int[]{1,2,3,4,5};
Integer[] arrInteger =new Integer[]{1,2,3,4,5};
```

## List<Integer> int[] Integer[] 相互转换

- List<Integer> --> int[] Integer[]

```java
//List<Integer> 转int[] Integer[]
int[] listToArr = list.stream()
        .mapToInt(Integer::valueOf)//转换为IntStream
        .toArray();
Integer[] listToArrInteger = list.toArray(new Integer[0]);
```

- int[] <--> Integer[]

```java
//int[] Integer[]相互转换
int[] IntegerArr2IntArr = Arrays.stream(arrInteger)//转换为Stream<Integer>
        .mapToInt(Integer::valueOf)//Stream<Integer>调用Integer::valueOf来转成IntStream
        .toArray();

Integer[] IntArr2IntegerArr = Arrays
        .stream(arr)//转换为IntStream
        .boxed()//int 装箱为 Integer
        .toArray(Integer[]::new);
```

- int[] Integer[] --> List<Integer>
  
```java
//int[] Integer[] 转换为 List<Integer>

//Integer[] 转换为 List<Integer>
ArrayList<Integer> listFromInteger = Arrays.stream(arrInteger)//转换为Stream<Integer>
        .collect(Collectors.toCollection(ArrayList::new));
ArrayList<Integer> listFromInteger2=new ArrayList<>(Arrays.asList(arrInteger));

//int[] 转换为 List<Integer> 可先转为Integer[]再转，也可在Stream中直接通过boxed()装箱，再转
List<Integer> listFromInt= Arrays.stream(arr)//转换为IntStream
        .boxed()//int 装箱为 Integer
        .collect(Collectors.toList());
```


## List<String> String[] 相互转换


- List<String> --> String[]

```java
List<String> list = new ArrayList<>();
list.add("a");
list.add("bb");
list.add("cda");
list.add("ddd");
list.add("ba");

String[] strArr = list.toArray(new String[0]);//new String[0] 创建一个长度为 0 的数组
//一定要加new String[0] 因为toArray()方法默认返回object[] 所以需要指定转换类型 
```

- String[] --> List<String> 

```java
List<String> list =Arrays.asList(new String("aa"),
        new String("bb"),
        new String("cc"),
        new String("dd"));
```

