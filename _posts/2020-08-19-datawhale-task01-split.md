---
layout: post
title: 'DataWhale Task01：分冶算法'
subtitle: '分冶刷题笔记'
date: 2020-08-19
author: Dave
tags: Java刷题
---

![](https://raw.githubusercontent.com/dendyikbc/PicGoBed/master/img/datawhale-ps.png)

>记录DataWhale 相对比较随意

### [169. 多数元素](https://leetcode-cn.com/problems/majority-element/)

- 先不管上来就计数 搭个hashmap的菜鸡解法

```java
import java.util.*;
class Solution {
    public int majorityElement(int[] nums) {
        //题目给的返回一个int 默认数组内只有一个满足
        //默认int 没有泛型
        //看样子 应该都是正的 其实自然数的范围 比较小
        int len=nums.length;
        int res=nums[0];//返回值默认
        Map<Integer,Integer> map=new HashMap<>(len);
        //Arrays.sort(nums);
        for(int i=0;i<len;i++){   
            if(map.containsKey(nums[i])){
                int count=map.get(nums[i]);
                map.put(nums[i],++count);
            }else{
                int cnt=0;
                map.put(nums[i],++cnt);
            }
        }//fori end
        

        Set<Map.Entry<Integer, Integer>> set = map.entrySet();
        // 遍历键值对对象的集合，得到每一个键值对的对象
        for (Map.Entry<Integer, Integer> me : set) {
            // 获取键和值
            int key = me.getKey();
            int value = me.getValue();
            if(value>len/2) res=key;
        }
        return res;
    }
}   
```
- 尝试分冶算法

```java
//没比hashmap快  主要看个思路
class Solution {
    public int majorityElement(int[] nums) {
        return getMajority(nums, 0, nums.length - 1);
    }

    /**
     * 递归分治地求解子数组的众数
     * @param 子数组以及子数组首尾元素的下标
     */
    private int getMajority(int[] a, int lo, int hi) {
        if(hi == lo)    return a[lo];
        int mid = lo + (hi - lo) / 2;
        int left = getMajority(a, lo, mid);
        int right = getMajority(a, mid + 1, hi);
        if(left == right)   return left;
        if(getCount(a, lo, mid, left) > getCount(a, mid + 1, hi, right))
            return left;
        else    return right;
    }

    /**
     * @return 子数组的众数在子数组中的出现次数
     */
    private int getCount(int[] a, int lo, int hi, int key) {
        int cnt = 0;
        for(int i : a)
            if(i == key)    
                cnt++;
        return cnt;
    }
}
```


- 看看扣友们的

```java
//惊奇 挺不错的
class Solution {
    public int majorityElement(int[] nums) {
        Arrays.sort(nums);
        return nums[nums.length/2];
    }
}

作者：pendygg
链接：https://leetcode-cn.com/problems/majority-element/solution/lan-de-xiang-yue-lai-yue-duo-luo-liao-by-pendygg/
来源：力扣（LeetCode）
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。
```

好的解答
- [Java-3种方法(计数法/排序法/摩尔投票法)](https://leetcode-cn.com/problems/majority-element/solution/3chong-fang-fa-by-gfu-2/)

- [使用栈来记忆](https://leetcode-cn.com/problems/majority-element/solution/shi-yong-zhan-lai-ji-yi-by-linxinfu/)

    >之前有个类似的也是用栈来解决，太久又忘记了



### [53. 最大子序和](https://leetcode-cn.com/problems/maximum-subarray/)


- 上来先dp搞起了

```java
class Solution {
    public int maxSubArray(int[] nums) {
        //一维dp搞起先 
        /*
        dp[i]表示以数组nums[i]结尾的子数组（0-i）的最大和为多少

        典型： dp[0]=0 
        dp[i] = Math.max(dp[i- 1] + nums[i], nums[i])    
        */
        int[] dp = new int[nums.length];
		dp[0] = nums[0];
		int max = nums[0];
		for (int i = 1; i < nums.length; i++) {
			dp[i] = Math.max(dp[i- 1] + nums[i], nums[i]);	
			if (max < dp[i]) {
				max = dp[i];
			}
		}
		return max;
    }
}  
```

- 分冶

> [整理一下看得懂的答案！！！](https://leetcode-cn.com/problems/maximum-subarray/solution/zheng-li-yi-xia-kan-de-dong-de-da-an-by-lizhiqiang/)

```java
class Solution {
public int maxSubArray(int[] nums) {
    return maxSubArrayDivideWithBorder(nums, 0, nums.length-1);
}

private int maxSubArrayDivideWithBorder(int[] nums, int start, int end) {
    if (start == end) {
        // 只有一个元素，也就是递归的结束情况
        return nums[start];
    }

    // 计算中间值
    int center = (start + end) / 2;
    int leftMax = maxSubArrayDivideWithBorder(nums, start, center); // 计算左侧子序列最大值
    int rightMax = maxSubArrayDivideWithBorder(nums, center + 1, end); // 计算右侧子序列最大值

    // 下面计算横跨两个子序列的最大值

    // 计算包含左侧子序列最后一个元素的子序列最大值
    int leftCrossMax = Integer.MIN_VALUE; // 初始化一个值
    int leftCrossSum = 0;
    for (int i = center ; i >= start ; i --) {
        leftCrossSum += nums[i];
        leftCrossMax = Math.max(leftCrossSum, leftCrossMax);
    }

    // 计算包含右侧子序列最后一个元素的子序列最大值
    int rightCrossMax = nums[center+1];
    int rightCrossSum = 0;
    for (int i = center + 1; i <= end ; i ++) {
        rightCrossSum += nums[i];
        rightCrossMax = Math.max(rightCrossSum, rightCrossMax);
    }

    // 计算跨中心的子序列的最大值
    int crossMax = leftCrossMax + rightCrossMax;

    // 比较三者，返回最大值
    return Math.max(crossMax, Math.max(leftMax, rightMax));
    }

}  
```

### [50. Pow(x, n)](https://leetcode-cn.com/problems/powx-n/)

- 第一眼我居然来暴力？ 当然是超时了

```java
//反面教材 暴力 不可取
class Solution {
    public double myPow(double x, int n) {
        if(n==0) return (double)1;
        double res=1.0;
        if(n>0){
            for(int i=0;i<n;i++){
                res*=x;
            }

        }else{
            for(int i=0;i<Math.abs(n);i++){
                res*=x;
            }
            res=1.0/res;
        }
        return res;
    }
}  
```
- 分冶的思考

>主要根据 幂的性质 和 分奇数偶数分别看待

![](https://cdn.mathpix.com/snip/images/BB5kE5PYqBEtvvFrwc_r6AVJKmMvfgF4BtJx4Wfq5oA.original.fullsize.png)

```java
class Solution {
    public double myPow(double x, int n) {
        if(x == 0.0f) return 0.0d;
        long b = n;
        double res = 1.0;
        if(b < 0) {
            x = 1 / x;
            b = -b;
        }
        while(b > 0) {
            if((b & 1) == 1) res *= x;//位运算 优化 取余操作 与hashmap源码那里类似   
            x *= x;
            b >>= 1;//位运算
        }
        return res;
    }
}
```
别人写的 可以速览

- [50. Pow(x, n) （快速幂，清晰图解）](https://leetcode-cn.com/problems/powx-n/solution/50-powx-n-kuai-su-mi-qing-xi-tu-jie-by-jyd/)
