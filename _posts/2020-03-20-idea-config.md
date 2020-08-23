---
layout: post
title: 'IntelliJ IDEA|车轮滚滚'
subtitle: 'idea 轮子'
date: 2020-03-20
author: Dave
tags: Java
---


>**记录idea相关车轮 避免忘记的时候四处寻找 相对比较随意**

>默认win下

### 配置篇

- [IntelliJ IDEA详细配置和使用教程](https://blog.csdn.net/m_m254282520/article/details/78900238) 
- [IntelliJ IDEA 使用教程(2019图文版)](https://www.jianshu.com/p/9c65b7613c30)

- [IDEA——最全配置](https://blog.csdn.net/qq_36135928/article/details/90348725)
- [在IDEA中添加类注释与方法注释模板](https://www.jianshu.com/p/bf66c4923aa8)
- [IDEA安装Leetcode插件](https://blog.csdn.net/u010180815/article/details/104728115?utm_medium=distribute.pc_aggpage_search_result.none-task-blog-2~all~first_rank_v2~rank_v25-1-104728115.nonecase&utm_term=idea%E4%B8%AD%E9%85%8D%E7%BD%AEleetcode)

- [IDE 刷题工具 leetcode editor](https://zhuanlan.zhihu.com/p/60309695)

添加类注释模板
>打开**idea IDE**，进入**File->settings->Editor->File and Code Templates->Class**

添加以下内容：
- 模板1
```java
/**
* @ClassName : ${NAME}  //类名
* @Description : ${description}  //描述
* @Author : Dave  //作者
* @Date: ${YEAR}-${MONTH}-${DAY} ${HOUR}:${MINUTE}  //时间
*/
```
- 模板2
```java
/**
  * Copyright (C), 2015-${YEAR}, XXX有限公司
  * FileName: ${NAME}
  * Author:   ${USER}
  * Date:     ${DATE} ${TIME}
  * Description: ${DESCRIPTION}
  * History:
  * <author>          <time>          <version>          <desc>
  * 作者姓名           修改时间           版本号              描述
  */
  
#if (${PACKAGE_NAME} && ${PACKAGE_NAME} != "")
    package ${PACKAGE_NAME};
#end
 
/**
 * 〈一句话功能简述〉<br> 
 * 〈${DESCRIPTION}〉
 *
 * @author ${USER}
 * @create ${DATE}
 * @since 1.0.0
 */
public class ${NAME} {
 
}

```

**添加方法注释模板**

```java
**
* @Param $param$ //参数
* @description $description$  //描述
* @author Dave //作者
* @date $date$ $TIME$ //时间
* @return $return$  //返回值
* @throws $throws$  //异常
*/
```

使用规则：**快捷键(/前缀+Tab)快速生成注释**


### 插件
- [TranslationPlugin](https://github.com/YiiGuxing/TranslationPlugin)

- [阿里巴巴Java代码规约插件p3c-pmd](https://mp.weixin.qq.com/s/lBDe60LFpdXoi2xbA-L2Qg?)

### 使用篇

- [Intellij IDEA 使用教程 | 菜鸟教程](https://www.runoob.com/w3cnote/intellij-idea-usage.html)

- [IntelliJ IDEA使用教程（很全）](https://www.cnblogs.com/yjd_hycf_space/p/7483921.html)
    >新建Java web项目演示

- [Intellij IDEA 创建Web项目并在Tomcat中部署运行](https://www.cnblogs.com/yjd_hycf_space/p/7483549.html)

- [Tomcat的JVM和连接数设置 ](https://www.cnblogs.com/yjd_hycf_space/p/8000237.html)

- [IntelliJ IDEA 常用设置讲解](https://www.cnblogs.com/yjd_hycf_space/p/7930929.html)
- [IDEA快捷键整理（最详细的）](https://www.cnblogs.com/yjd_hycf_space/p/7479646.html)
### Deubg部分
- [IDEA调试总结（设置断点进行调试）](https://www.cnblogs.com/yjd_hycf_space/p/7483471.html)

### Spring/SpringBoot
- [IntelliJ IDEA搭建Spring环境](https://blog.csdn.net/cflys/article/details/70598903)
- [使用IDEA搭建一个简单的SpringBoot项目——详细过程](https://blog.csdn.net/baidu_39298625/article/details/98102453)
- [idea快速搭建springboot项目](https://www.cnblogs.com/yybrhr/p/9811951.html)


- []()
- []()

### idea中JDK源码的正确打开方式
- [IDEA查看JDK源代码](https://www.cnblogs.com/lbrs/p/10923748.html)
- [IDEA查看Java源码技巧](https://blog.csdn.net/qq_28666081/article/details/83898684)

```java
1 查看接口的实现类：Ctrl+Alt+B
    选中按快捷键，然后跳到实现类的地方去

2 返回上/下个光标地方:Alt+<- 和 Alt+->
    可通过修改快捷键(搜关键字left、right)找到对应并改为 Ctrl+J

3 查看Java方法调用树(被调/主调)：Ctrl+Alt+H
    分为调用当前方法的树、当前方法调用的下级方法

4 查看类继承关系图：Ctrl+Alt+U
5 查看当前类的继承树：Ctrl+H
6 查看定义的变量在哪里被调用：Ctrl+Alt+F7
7 查看一个类中有什么方法：Alt+7 或 点左侧边栏Structure
```

### idea的偷懒迷惑行为
# How to use intellij idea more efficient in daily coding？
>This is my own tips in the process of my java learning with intellij idea，not stand any others's choice.
>Keep Updating 

```java
//ctrl + 点击名字 可以来到源代码
//ctrl + 字母n 打开搜索框 按string 也打开源代码

System.out.print("=================");//没有回车
System.out.println("=================");//有回车

```
# Part.01 Quick Code 

## Quick code to generate the method for one standard java class(java bean)
Code->Generate

+ ->Constructor
    - none
    - all
+ ->Getter&Setter
    - all


## Quick code to generate the main method in the program
`psvm` + `ENTER`
```java
  public static void main(String[] args) {
        
    }
```

## Quick code to generate the loop in the program

`fori` + `ENTER`
```java

        for (int i = 0; i < ; i++) {
            
        }

```

`bytes.fori` + `ENTER`
```java


```


`100.fori` + `ENTER`
```java

        for (int i = 0; i < 100; i++) {
            
        }

```

`itar` + `ENTER`

```java

        for (int i = 0; i < array.length; i++) {
            Person person = array[i];

        }

```

## Quick code to generate the `System.out.println()` in the program

`sout` + `ENTER`
```java
System.out.println();
```

`100.sout` + `ENTER`
```java
System.out.println(100);
```
`"dave".sout` + `ENTER`
```java
System.out.println("dave");
```
