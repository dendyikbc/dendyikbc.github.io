---
layout: post
title: '数组问题：差分思想与前缀和思想'
subtitle: ''
date: 2021-10-10
author: Dave
cover: ''
tags: 算法与刷题
---



## 数组问题：差分思想与前缀和思想

> 数组问题中的辅助数组思想
>
> - 差分思想
> - 前缀和思想

### 理论

**差分数组**：对于已知有n个元素的数组a，建立记录它每项与前一项差值的差分数组diff，显然，diff[0] = a[0] - 0=a[0];对于整数i∈[1,n)，我们让diff[i] = a[i] - a[i-1]。

**前缀和数组**： 前缀和定义为从数组第一位数到当前位置这个区间内的所有的数字之和；对于已知有n个元素的数组a，建立记录它每项到数组首部区间所有数字之和的前缀和数组preSum，preSum[0] = 0，对于整数i∈[1,n)，preSum[i] = preSum[i-1] + a[i-1]。

![数组思想](/Users/zhouyun/Documents/picc0lo.top/数组思想.png)

**性质**

```
差分数组 
diff[0] = a[0] - 0
diff[1] = a[1] - a[0]
......
diff[i] = a[i]-a[i-1]   （i>=1）

原数组与差分数组关系（差分数组还原原数组） 
a[0] = diff[0]
a[1] = diff[1] + a[0] = diff[0] + diff[1]
......
a[i] = diff[0] + diff[1]+......+ diff[i]

前缀和数组
preSum[0] = 0
preSum[1] = preSum[0] + a[0]
......
preSum[i] = preSum[i-1] + a[i-1]   （i>=1）

结论：
1）原数组其实为其差分数组的前缀和数组；
2）由前缀和推导原数组：a[i] = preSum[i+1] - preSum[i] i∈[0,n)
3）由差分数组推导前缀和：
preSum[k] = a[0] + a[1] +......+ a[k-1]
					= diff[0] + (diff[0] + diff[1]) +......+ (diff[0] + diff[1]+......+ diff[k-1])
					= k*diff[0] + (k-1)*diff[1]+......+ 1*diff[k-1] k∈[1,n]
```

**结论3表达式**
$$
\operatorname{preSum}_{k}=\sum_{i=0}^{k-1} a_{i}=\sum_{i=0}^{k-1} \sum_{j=0}^{i} diff_{j}=\sum_{i=0}^{k-1}(k-i) \cdot diff_{i}
$$


### 运用

- 快速区间统一操作

```
原数组与差分数组关系（差分数组还原原数组） 
a[0] = diff[0]
a[1] = diff[1] + a[0] = diff[0] + diff[1]
......
a[i] = diff[0] + diff[1]+......+ diff[i]

diff[0] = diff[0]+1,则index >= 0 的数值均加一，同步影响范围[0,n)；
diff[2] = diff[2]-1,则index >= 2 的数值均减一，同步影响范围[2,n)；
结果：[0,1] 区间 加一

因此，对[i,j]区间进行统一加k，只需diff[i]加k，再对diff[j+1]减k。减法类比。
```

- 区间求和操作

  > 由前缀和推导原数组：a[i] = preSum[i+1] - preSum[i] i∈[0,n)

```
a[i] = preSum[i+1] - preSum[i] i∈[0,n)
```



### 试练



