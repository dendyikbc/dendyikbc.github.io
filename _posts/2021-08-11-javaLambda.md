---
layout: post
title: 'Java中Lambda使用笔记'
subtitle: 'javaLambda'
date: 2021-08-11
author: Dave
cover: ''
tags: Java
---

## Lambda Java

>最近回顾一下，写个笔记。

**最前面**

>Lambda 允许把函数作为一个方法的参数（函数作为参数传递进方法中）。使用 Lambda 表达式可以使代码变的更加简洁紧凑。

Lambda语法范式

```java
(parameters) -> expression
//或
(parameters) ->{ statements; }
```

### 基础示例

```java
// 1. 不需要参数,返回值为 5  
() -> 5  
  
// 2. 接收一个参数(数字类型),返回其2倍的值  
x -> 2 * x  
  
// 3. 接受2个参数(数字),并返回他们的差值  
(x, y) -> x – y  
  
// 4. 接收2个int型整数,返回他们的和  
(int x, int y) -> x + y  
  
// 5. 接受一个 string 对象,并在控制台打印,不返回任何值(看起来像是返回void)  
(String s) -> System.out.print(s)
```

### 典型示例

- 比较器写法
  
>- 比较器写法主要有名内部类写法与Lambda表达式写法。
>- 主要为定制排序，常见的有：字符串长度排序，字符串首字母排序，字符串结尾字母排序，主要与`Arrays.sort()`与`stream().sorted()`结合使用。

```java
import java.util.ArrayList;
import java.util.Arrays;
import java.util.Comparator;
import java.util.List;

public class testLambda {
    public static void main(String[] args) {
        List<String> list = new ArrayList<>();
        list.add("piccolo");
        list.add("dave");
        list.add("dava");
        list.add("mia");
        list.add("maybe");

        List<Integer> numList = Arrays.asList(1,2,3,4,5);

        String[] strArrays = {"piccolo",
                "dave",
                "dava",
                "mia",
                "maybe"};

        // 1).字符串长度 增序输出
        System.out.println("比较器定制排序：字符串长度增序");

        System.out.println("### Arrays.sort() 相同长度不做顺序变化");
        Arrays.sort(strArrays, (String s1, String s2) -> (s1.length() - s2.length()));
        Arrays.stream(strArrays).forEach(System.out::println);

        System.out.println("### Arrays.sort() 相同长度按照字符串大小增序");
        Arrays.sort(strArrays, (String s1, String s2) ->
                ((s1.length() - s2.length())==0?s1.compareTo(s2):(s1.length() - s2.length()))
        );
        Arrays.stream(strArrays).forEach(System.out::println);

        System.out.println("### stream().sorted() 相同长度按照字符串大小增序");
        list.stream().sorted((s1,s2) -> {
            int num = s1.length()-s2.length();
            int num2 = num==0?s1.compareTo(s2):num;//s1.compareTo(s2) s1大于s2 返回1，反之，返回-1  这里对同等长度的还比较了一下字符大小
            return num2;//num2 为正时，s1比s2长，返回为正，表示按长度增序
        }).forEach(System.out::println);
        System.out.println("++++++++++++++++");
        
    }
}
```

补充；其余比较类型示例

```java
//匿名内部类写法替代lambda写法
Arrays.sort(strArrays, new Comparator<String>() {
    @Override
    public int compare(String s1, String s2) {
        return (s1.length() - s2.length());
    }
});

// 其余比较类型用例
//按照字符串大小增序
Comparator<String> sortByStringValue = (String s1, String s2) -> (s1.compareTo(s2)); 
//按照字符串最后一个字符大小增序
Comparator<String> sortByLastCharValue = (String s1, String s2) -> (s1.charAt(s1.length() - 1) - s2.charAt(s2.length() - 1));  
```

- 其余操作举例

```java
//实现Runnable接口

// 1使用匿名内部类  
new Thread(new Runnable() {  
    @Override  
    public void run() {  
        System.out.println("Hello Thread !");  
    }  
}).start();  
// 2使用 lambda expression  
new Thread(() -> System.out.println("Hello Thread !")).start();  
  
// 1使用匿名内部类  
Runnable race1 = new Runnable() {  
    @Override  
    public void run() {  
        System.out.println("Hello Runnable  !");  
    }  
};   
// 2使用 lambda expression  
Runnable race2 = () -> System.out.println("Hello Runnable  !");  
   
// 直接调用 run 方法(没开新线程)  
race1.run();  
race2.run();  
```

