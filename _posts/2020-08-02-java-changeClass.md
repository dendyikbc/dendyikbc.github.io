---
layout: post
title: 'Java 基本数据类型转换'
subtitle: 'String-char-int'
date: 2020-08-02
author: Dave
tags: Java 
---

>记录基本数据类型的转换

### char与String相互转换

```java
    //Sting -> char[]
    String str = “abcdefg”；
    char[] ch = str.toCharArray();
    //Sting -> char
    char b=str.charAt(0);//a


    //char -> String
    //核心为String.valueOf(char)
    char c='c';
    //1.
    String s = String.valueOf(c); //效率最高
    //2.
    String s = Character.toString(c);// Character.toString(char)方法实际上直接返回String.valueOf(char)
    //3.
    String s = new Character(c).toString();
    //4.
    String s = "" + c;// 用+号做连接符，虽然方法简单，但这个效率最低；
    //5.
    String s = new String(c);//String的构造方法实现

    //char[] -> String
    char[] chararr=new char[]{'1','0','1','0'};
    //1.
    String s0 = String.valueOf(chararr); //String.valueOf(char[])
    //2.
    String s1 = new String(chararr);//String的构造方法实现

    System.out.println(s0);
    System.out.println(s1);



```
### String与StringBuffer之间的转换

>在这些操作上StringBuilder=StringBuffer

```java
    //String -> StringBuffer
    String str = "Hi Dave!";
    //方式一：构造方法
    StringBuffer buffer = new StringBuffer(str);
    //方式二：通过append方法
    StringBuffer buffer2 = new StringBuffer();
    buffer2.append(str);

    //StringBuffer -> String
    StringBuffer buffer3 = new StringBuffer();
    buffer3.append("Hi Mia!");
    //方式一：通过构造方法
    String str2 = new String(buffer3);
    //方式二：通过toString方法
    String str3 = buffer3.toString();
    //方式三：通过截取子字符串将StringBuffer转换为String
    String str4 = sb.substring(0, buffer3.length());		
```

### char与int之间的转换

```java

    //char -> int
    System.out.println((int) '2');//50  char本身的ansii值
    /*
    往往我们想得到的是
    输入：char '1'
    输出：int 1
    */
    char ch = '9';
    //1.
    if (Character.isDigit(ch)){  //借助String-> int 的Integer.parseInt()方法
	int num = Integer.parseInt(String.valueOf(ch));
	System.out.println(num);
    }
    //2.
    if (Character.isDigit(ch)){  //借助ANSII值做减法
	int num = (int)ch - (int)('0');
	System.out.println(num);
    }

    //int -> char
    /*
    形如
    输入：int 1
    输出：char '1'
    */

    int num= 1;
    char c0 = Character.forDigit(num,10);//Character类自有的转换方法
    char c1 = String.valueOf(num).charAt(0);//int-> String-> char 曲线救国转换
    char c2 = (char)('0' + num);//利用char类可以直接与int类 做相加
    char c3 = (char)(48+ num);//等同与c2，利用char类可以直接与int类 做相加，这里用ANSII码
    System.out.println(c0);
    System.out.println(c1);
    System.out.println(c2);
    System.out.println(c3);
    //int -> char[]
    /*
    形如
    输入：int 1234
    输出：char[] {'1','2','3','4'}
    */
    int num= 1010;//作二进制时表示10，作十进制表示1010，作八进制时表示520，作十六进制时表示4112
    //二进制 1010 真值：10
    //十进制 1010 真值：1010
    //八进制 1010 真值：520
    //十六进制 1010 真值：4112

    //int -> String -> char[]
    String str = Integer.toString(num);//int -> String 的三个方法都ok
    char[] chararr = str.toCharArray();
    for(int i:chararr){//输出数组chararr
        System.out.print((char)i+" ");//1 0 1 0 
        //System.out.print(i+" ");//49 48 49 48 
    }
    System.out.println("");

    //int 1010 -> char[]{'1','0','1','0'} -> int[]{1,0,1,0} 
    int[] intarr=new int[chararr.length];
    for (int i = 0; i < chararr.length; i++) {
         intarr[i] = (int)chararr[i] - (int)('0');//这里利用char -> int的两个方法都可   
    }
    for(int i:intarr){//输出数组chararr
        System.out.print(i+" ");//1 0 1 0 
    }
```

### String与int之间的转换

```java
    //String -> int
    String str="1010";
    int num0=Integer.parseInt(str);//直接使用静态方法，不产生多余对象，但会抛出异常
    int num1=Integer.parseInt(str,2);//按照二进制转换 输出用十进制 1010b=10d 所以输出 10
    int num2=Integer.valueOf(str).intValue();//Integer.valueOf(s) 相当于 new Integer(Integer.parseInt(s))，也会抛异常，但会多产生一个对象
    System.out.println(num0);
    System.out.println(num1);
    System.out.println(num2);

    //String -> int[]
    //"1010" -> int[]{1,0,1,0}
    String str = "1010";
    int[] arr = new int[str.length()];
    for (int i = 0; i < str.length(); i++) {
        arr[i] = Integer.parseInt(str.substring(i, i + 1));//substring [i,i+1) 左闭右开 
    }
    for(int i:arr){//输出数组arr
        System.out.print(i+" ");
    }

    //int -> String 
    int num= 1010;
    String s0 = String.valueOf(num);//String的静态方法，只产生一个对象
    String s1 = Integer.toString(num);
    String s2 = "" + num;//会产生两个String对象
    System.out.println(s0);
    System.out.println(s1);
    System.out.println(s2);
    //int[] -> String 
    int[] arr = new int[]{1,0,1,0};
    StringBuffer buffer = new StringBuffer();
    for (int i = 0; i < arr.length; i++) {
        buffer.append(String.valueOf(arr[i]));//这里利用int -> String的三个方法都可  
    }
    System.out.println(buffer);//1010

    //int[] -> char[] -> String
    int[] arr = new int[]{1,0,1,0};
    char[] chararr=new char[arr.length];
    for (int i = 0; i < arr.length; i++) {
         chararr[i] = Character.forDigit(arr[i],10);//这里利用int -> char的四个方法都可   
    }
    String str = String.valueOf(chararr); //char数组转换成String
    //String str = new String(chararr); //char数组转换成String 也可以用 String的构造方法
    System.out.println(str);//1010
```

2020/08/12 补充
一个很基础的，今天犯错了，牛客网因为这个没有调试出来；

```java
/*
题目要求输出 4 9 两个结果用空格隔开，我写的
System.out.println(1+' '+2);
死活不对，一直0%，其实本来很简单一个问题

最后改为如下 即可
System.out.println(1+" "+2);

*/
        System.out.println(1+' '+2);//' ' 的ANSII 值为32 这里 执行 1+32+2 输出35
        //如果想要输出：1 2 （1 空格 2）这种形式  应当改为如下 即可避免 空格字符''' 进行加法运算
        //今天牛客死活调不出来，就是这里出了问题  
        System.out.print(1);
        System.out.print(' ');
        System.out.println(2);
        //或者如下
        System.out.println(1+" "+2);
        //具体原因如下：+号两侧有一侧为String时，+才是连接符；char与int时 执行加法操作 
        System.out.println(1+2);//3
        System.out.println('1'+'2');//ANSII值相加 49+50=99 
```
