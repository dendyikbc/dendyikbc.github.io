---
layout: post
title: 'lc60.排列序列引出的：康托与逆康托'
subtitle: ''
date: 2022-06-01
author: Dave
cover: ''
tags: 算法与刷题
---

## lc60.排列序列引出的：康托与逆康托

#### [60. 排列序列](https://leetcode.cn/problems/permutation-sequence/)

### 解题思路

首先，排列序列呈现出明显的分段模式，如图
![图解lc60-1 分段示意drawio.png](https://pic.leetcode-cn.com/1658508650-uOAtml-%E5%9B%BE%E8%A7%A3lc60-1%20%E5%88%86%E6%AE%B5%E7%A4%BA%E6%84%8Fdrawio.png)

```
对于n的序列(本质为全排列树的叶子节点)
1）segment_1类型的段 有 n 个，每段有(n-1)!；

2）对段内持续分段，直至未使用元素仅存一个时，到达最小段划分；

3）每段内的最后一个序列，保持特点：下一元素为剩余元素最大数；

 实例说明：

一级分段，segment_1，分段步长 3！= 6；

二级分段，segment_1_1，分段步长2！= 2；

三级分段，segment_1_1_1，分段步长 1！= 1；

对于segment_1，均为以1起始序列，段尾序列1432，1的下一元素为剩余元素最大数4；
对于segment1_1，均为以12起始序列，段尾序列1243，12的下一元素为剩余元素最大数4；

```

#### `所以很自然想到取模进行分段选择，留余进行递推到定位到最小段（即一个叶子节点）`

```java
//伪代码
while(没定位到叶子节点){
a = k/step;
b = k%step;
//......（特殊逻辑选择当前元素）
k=b;
}


//a为模 b为余 step 为每一次分段步长
/*
特殊逻辑：
1) a=0,b!=0时 需要找到此时第1小数；
    a为0，b非0，表示本次分段步长比k大，第k个序列落在本次分段内，当前元素选择剩余最小数；
2) a=0,b=0时 需要找到此时剩余最大数；
    a为0，b为0，递推k为0，表示前一次分段刚好完整结束，第k个序列命中当前段尾，当前元素选择剩余最大数；
3) a!=0,b!=0时 找第a+1小数
    b非0，表示本次分段步长不够，需跨入下一段，当前元素选择剩余第a+1小数；
4) a!=0,b=0时 找第a小数
    b为0，表示本次分段刚好完整结束，不能跨入下一段，当前元素选择剩余第a小数；
*/
```

#### 分段选取的逻辑有没有看着很复杂？xdm 

![图解lc60.png](https://pic.leetcode-cn.com/1658512474-HxnnQJ-%E5%9B%BE%E8%A7%A3lc60.png)




> 这里主要是 `k自减1` 的引入


```
上述逻辑简化，首先3）是覆盖1）的；
其次，对于4）本次分段正好完整结束，k自减1 的操作，正好可以使原来的4）退化为 3），
举例，n=4，k=4，如图过程。
到这里，1）、3）、4）得到统一，重点看2）。

举例，n=4，k=4，如图过程。
步骤3，出现a=b=0,即上一步后递推k为0，前一次分段刚好完整结束，命中当前段尾，当前元素理应选择剩余最大数，
k-1引入后，对于步骤2整除现象，递推k = 2！-1，它的影响在步骤3中呈现为a =（2！-1）/1！ = 2 - 1 ，注意减1的出现，在步骤3以a+1原则去寻找剩余2个数中第a+1小数时，变为寻找最大数。

对于原步骤m，出现整除现象时，k-1引入后，步骤m，a减1，b = (n-m)!-1, 它的影响在步骤m+1中呈现为a = （(n-m)!-1)/(n-m-1)! = n-m-1，在第m+1步骤中以a+1原则去寻找剩余的n-m个数字中第n-m小数，即为寻找最大数，与2）匹配。

注意：(n!-k)/(n-1)! = n - 1 ,其中 (n-1)! >= k >= 1

其实，本质上k-1的引入，在原本步骤没有整除的时候，均出现余数减1；
在整除的时候，出现商减1，余数为分段大小减1，导致的下一步骤的商与剩余数字的个数存在关系。
//伪代码
k=k-1//初始化，最前面

while(没定位到叶子节点){
a = k/step;
b = k%step;
//......（特殊逻辑选择当前元素）
k=b;
}

//a为模 b为余 step 为每一次分段步长
/*
特殊逻辑：
当前元素选择剩余第a+1小数；
*/
```


### 代码

####  用k做递推

```java
class Solution {
    int n, k;
    boolean[] visit;
    // n!  t[n] n的使用 visit[n]
    int[] t = new int[]{1, 1, 2, 6, 24, 120, 720, 5040, 40320};

    public String getPermutation(int n, int k) {
        // 初始化
        this.n = n;
        this.k = k;
        visit = new boolean[n + 1];
        StringBuilder str = new StringBuilder();
        // 搜索
        for (int i = 1; i <= n; i++) {
            search(str);
        }
        return str.toString();
    }

    private void search(StringBuilder str) {
        if (str.length() == n) {
            return;
        }
        int a = k / t[n - 1 - str.length()];
        int b = k % t[n - 1 - str.length()];
        int target = 0;
        if(a==0){
            if(b!=0){
                target = 1;
            }else{
                target = n - str.length();
            }
        }else{
            if(b==0){
                target = a;
            }else{
                target = a+1;
            }
        }
        for (int i = 1; i <= n; i++) {
            if (!visit[i]) {
                --target;
                if (target == 0) {
                    str.append(i);
                    visit[i] = true;
                    break;
                }
            }
        }//fori end
        k = b;
    }
}
```

####  用k-1做递推

```java
class Solution {
    int n, k;
    boolean[] visit;
    // n!  t[n] n的使用 visit[n]
    int[] t = new int[]{1, 1, 2, 6, 24, 120, 720, 5040, 40320};

    public String getPermutation(int n, int k) {
        // 初始化
        this.n = n;
        this.k = k - 1;
        visit = new boolean[n + 1];
        StringBuilder str = new StringBuilder();
        // 搜索
        for (int i = 1; i <= n; i++) {
            search(str);
        }
        return str.toString();
    }

    private void search(StringBuilder str) {
        if (str.length() == n) {
            return;
        }
        int a = k / t[n - 1 - str.length()] + 1;
        int b = k % t[n - 1 - str.length()];
        for (int i = 1; i <= n; i++) {
            if (!visit[i]) {
                --a;
                if (a == 0) {
                    str.append(i);
                    visit[i] = true;
                    break;
                }
            }
        }//fori end
        k = b;
    }
}
```



### 康托与逆康托

> 写完了去找官解的评论发现，k-1递推实际就是逆康托。
>
> 为啥k要减1？康托展开取值范围是从0开始？
>
> 解答：递推过程除法过程中余数范围为：0 ～ (n-m)!-1，前述特殊逻辑一直在处理整除情况导致的跨段与不跨段判定问题。
>
> 原题意:对n的序列编号[1,n!]，取排列第k个序列，但对于第一次取模分段起，余数的范围均为[0,(n-m)!-1]，0始终需特殊处理；
>
> 若转换为 对n的序列编号[0,n!-1]，取排列第k-1个序列，则与余数范围存在对应，递推过程中，可以统一处理逻辑。

- [康托展开和逆康托展开](https://blog.csdn.net/wbin233/article/details/72998375)

  > 康托展开：序列  到 序号
  >
  > 
  > 逆康托展开：序号 到 序列 

例如有3个数（1，2，3），则其排列组合及其相应的康托展开值如下：

| 排列组合 | 名次 | 康托展开 | 值   |
| -------- | ---- | -------- | :--: |
|123	|1	|0 * 2! + 0 * 1! + 0 * 0!	|0|
|132|2	|0 * 2! + 1 * 1! + 0 * 0!	|1|
|213	|3	|1 * 2! + 0 * 1! + 0 * 0!|	2|
|231	|4|1 * 2! + 1 * 1! + 0 * 0!	|3|
|312	|5	|2 * 2! + 0 * 1! + 0 * 0!|	4|
|321	|6|	2 * 2! + 1 * 1! + 0 * 0!|	5|