---
layout: post
title: 'DataWhale Task03：查找之 查找表'
subtitle: '查找表刷题笔记'
date: 2020-08-25
author: Dave
tags: Java刷题
---

![](https://raw.githubusercontent.com/dendyikbc/PicGoBed/master/img/datawhale-ps.png)

>记录DataWhale 相对比较随意


**查找表的基本三类结构以及数据结构**

- **第一类:只查找有无-set**
    - 元素a是否存在,通常用set集合
    - se只存储键,而不需要对应其相应的值。
    - se中的键不允许重复
- **第二类:需查找重复次数**
    - 元素a出现了几次的结果返回:**推荐 LinkedList ArrayDeque**

        ```java
        //LinkedList ArrayDeque有此方法
        //ArrayList 无此方法
        removeFirstOccurrence(Object o) //删除第一次出现的指定元素
        removeLastOccurrence(Object o) //删除最后一次出现的指定元素
        ```
    - 元素a出现了几次的次数返回:**推荐 HashMap**

        ```java
        //key存放元素
        //value存放重复次数
        ```
- **第三类:改变映射关系-map**
    - 通过将原有序列的关系映射统表示为其他


### [349. 两个数组的交集](https://leetcode-cn.com/problems/intersection-of-two-arrays/)

- 两个set 两次遍历 只记录 有无 不记录 重复几次
```java
class Solution {
    public int[] intersection(int[] nums1, int[] nums2) {
        HashSet<Integer> set1=new HashSet<>();
        HashSet<Integer> ans=new HashSet<Integer>();     
        for(int i:nums1){
            set1.add((Integer)i);
        }
        for(int i:nums2){
            if(set1.contains(i)){
                //todo
                ans.add(i);
            }
        }
        int[] res=new int[ans.size()];
        int index=0;
        for(Integer i:ans){
            res[index++]=i;
        }
        return res;
    }//method end
}//class end
```
### [350. 两个数组的交集 II](https://leetcode-cn.com/problems/intersection-of-two-arrays-ii/)

- 在上一题基础上记录重复次数

     ```java
        //leetcode上大家都喜欢用hashmap，可能是参考官方解法比较多，但是这里用LinkedList/ArrayDeque 明显简洁很多

        //LinkedList ArrayDeque都有此方法
        //注意：ArrayList 无此方法
        removeFirstOccurrence(Object o) //删除第一次出现的指定元素
        removeLastOccurrence(Object o) //删除最后一次出现的指定元素
    ```

```java
class Solution {
    public int[] intersect(int[] nums1, int[] nums2) {
        LinkedList<Integer> arr1=new LinkedList<Integer>();
        ArrayList<Integer> ans=new ArrayList<Integer>();
        for(int i:nums1){
            arr1.add(i);
        }
        for(int i:nums2){
            if(arr1.contains(i)){
                //todo
                ans.add(i);
                arr1.removeFirstOccurrence(i);
            }
        }
        int[] res=new int[ans.size()];
        int index=0;
        for(Integer i:ans){
            res[index++]=i;
        }
        return res;
 
    }//method end
}//class end

```
### [242. 有效的字母异位词](https://leetcode-cn.com/problems/valid-anagram/)

- 简单的一维dp 而且 连续的约束条件 状态转移异常简单 并且 输出状态 只是记录最大值 无需返回该具体状态
```java

```
### [349. 两个数组的交集](https://leetcode-cn.com/problems/intersection-of-two-arrays/)

- 简单的一维dp 而且 连续的约束条件 状态转移异常简单 并且 输出状态 只是记录最大值 无需返回该具体状态
```java

```
### [349. 两个数组的交集](https://leetcode-cn.com/problems/intersection-of-two-arrays/)

- 简单的一维dp 而且 连续的约束条件 状态转移异常简单 并且 输出状态 只是记录最大值 无需返回该具体状态
```java

```
### [349. 两个数组的交集](https://leetcode-cn.com/problems/intersection-of-two-arrays/)

- 简单的一维dp 而且 连续的约束条件 状态转移异常简单 并且 输出状态 只是记录最大值 无需返回该具体状态
```java

```
### [349. 两个数组的交集](https://leetcode-cn.com/problems/intersection-of-two-arrays/)

- 简单的一维dp 而且 连续的约束条件 状态转移异常简单 并且 输出状态 只是记录最大值 无需返回该具体状态
```java

```

**车轮滚滚 时间**

五分钟学算法的动态规划系列:
- [浅谈什么是动态规划以及相关的「股票」算法题](https://mp.weixin.qq.com/s?__biz=MzUyNjQxNjYyMg==&mid=2247485288&idx=1&sn=fd043fc723f38bcaecc90d9945981f8a&chksm=fa0e68e9cd79e1ffd965205bb06b1731539bf2e0bbc5991664f5d1d9721b346ec08c85bb9042&scene=21#wechat_redirect)

