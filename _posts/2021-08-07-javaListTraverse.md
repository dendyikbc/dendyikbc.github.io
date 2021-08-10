---
layout: post
title: ' Java中List的遍历'
subtitle: 'javaListTraverse'
date: 2021-08-07
author: Dave
cover: ''
tags: Java
---


**最前面**
- [Java 在线工具](https://c.runoob.com/compile/10)

```java
import java.util.ArrayList;
import java.util.Iterator;
import java.util.List;

public class ListTraverse {
    public static void main(String[] args) {
        List<String> list = new ArrayList<>();
        list.add("piccolo");
        list.add("dave");
        list.add("mia");
        list.add("maybe");

        // 1.普通for循环遍历
        // 性能最佳
        System.out.println("第1种遍历方式：普通for循环");
        for (int i = 0; i < list.size(); i++) {
            System.out.println(list.get(i));
        }
        System.out.println("-------------------------------------");

        // 2.迭代器遍历
        // 需要注意的是Iterator对象的remove方法是迭代过程中删除元素的唯一方法
        System.out.println("\n第2种遍历方式：迭代器");
        Iterator<String> iterator = list.iterator();
        while (iterator.hasNext()) {
            String str = iterator.next();
            System.out.println(str);
        }
        System.out.println("-------------------------------------");

        // 3.增强for循环遍历
        // 内部为迭代器
        System.out.println("\n第3种遍历方式：增强for循环");
        for (String item : list) {
            System.out.println(item);
        }
        System.out.println("-------------------------------------");

        // 4.Lambda 表达式遍历（JDK 1.8）
        // forEach内部为增强for
        System.out.println("\n第4种遍历方式：Lambda 表达式遍历");
        list.forEach(one -> {
            System.out.println(one);
        });
        System.out.println("-------------------------------------");

        // 4.Lambda 表达式遍历（JDK 1.8）
        System.out.println("\n第4种遍历方式：Lambda 表达式遍历");
        list.forEach(System.out::println);
        System.out.println("-------------------------------------");

        // 4.Lambda 表达式遍历（JDK 1.8）
        // 支持多线程并行
        System.out.println("\n第4种遍历方式：Lambda 表达式遍历");
        list.stream().forEach(System.out::println);
        System.out.println("-------------------------------------");


    }
}
```
