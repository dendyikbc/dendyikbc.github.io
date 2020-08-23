---
layout: post
title: 'DataWhale Task02：动态规划'
subtitle: '动态规划刷题笔记'
date: 2020-08-22
author: Dave
tags: Java刷题
---

![](https://raw.githubusercontent.com/dendyikbc/PicGoBed/master/img/datawhale-ps.png)

>记录DataWhale 相对比较随意


动态规划四步解题法
- **1.确定状态**
    - 确定最优策略的最后一步
    - 转化为子问题
    - 除了过程中状态外，最后输出状态也需要确定

- **2.转移方程**
    - 根据子问题得到
        >往往可以画格子

 - 3.**初始条件和边界条件**
     - 需要考虑周全
 - 4.**注意计算顺序**
     - 主要是 记忆化存储 往往具备一个顺序

- 5.优化角度(非必须)
    - 时间复杂度 空间复杂度 优化
        >一个典型为 有时可将二维dp优化为一维


### [674. 最长连续递增序列](https://leetcode-cn.com/problems/longest-continuous-increasing-subsequence/)

- 简单的一维dp 而且 连续的约束条件 状态转移异常简单 并且 输出状态 只是记录最大值 无需返回该具体状态
```java
//leetcode submit region begin(Prohibit modification and deletion)
class Solution {
    public int findLengthOfLCIS(int[] nums) {
        int len=nums.length;
        if(len<1) return 0;
        int[] dp=new int[len];
        for (int i = 0; i < len; i++) {
            dp[i]=1;
        }
        //这道题目是连续的，dp数组应该具备的初始值为1 因为 最低为1
        //dp[i]是以nums[i]这个数结尾的最长递增子序列的长度
        int res=dp[0];
        for(int i=1;i<len;i++){
            if(nums[i]>nums[i-1]){
                dp[i]=dp[i-1]+1;
                res=Math.max(dp[i],res);
            }
            else
                dp[i]=1;
                /*
                这里并不是dp[i]=dp[i-1];//没有连续的 约束 就为这个等式 但是这里有连续的约束
                [1 3 5 4 7]数组 
                dp[3]=1 因为以nums[3]结尾的最长连续子串为[4]
                dp[4]=2 因为以nums[4]结尾的最长连续子串为[4,7]
                */
        }
        return res;
    }
}
```

### [5. 最长回文子串](https://leetcode-cn.com/problems/longest-palindromic-substring/)

- 注意计算顺序；不难 这里dp的值，不代表长度，代表子串是否为回文串

```java
class Solution {
    public String longestPalindrome(String s) {
        if(s.length()<2) return s;
        
        int start=0;
        int end=0;
        char[] ch=s.toCharArray();
        boolean[][] dp=new boolean[ch.length][ch.length];
        for (int i = 0; i < ch.length; i++) {
            dp[i][i]=true;
        }//赋值子串[i,i]必为回文串
        
        int maxLen=1;
        for (int j = 1; j < ch.length; j++) {
            for (int i = 0; i < j; i++) {//计算顺序可以画格子 根据 画格子 j > i的前提下，i从小开始计算
                if(ch[i]==ch[j]){
                    if(j-i<3){
                        dp[i][j]=true;//aba  或 aa 都为回文子串
                    }else{
                        dp[i][j]=dp[i+1][j-1];
                    }
                }

                if(dp[i][j]){
                    if(j-i+1>maxLen){
                        maxLen=j-i+1;
                        start=i;
                        end=j;
                    }
                }

            }//forj end
        }//fori end
        return s.substring(start,end+1);
    }
}
```

- 遍历索引+中心扩散 

    除了枚举字符串的左右边界以外，比较容易想到的是**枚举可能出现的回文子串的“中心位置”，从“中心位置”尝试尽可能扩散出去，得到一个回文串**。

    因此中心扩散法的思路是：遍历每一个索引，以这个索引为中心，利用“回文串”中心对称的特点，往两边扩散，看最多能扩散多远。

    枚举“中心位置”时间复杂度为 *O(N)*，从“中心位置”扩散得到“回文子串”的时间复杂度为 *O(N)*，因此时间复杂度可以降到 *O(N^2)*。

    在这里要注意一个细节：回文串在长度为奇数和偶数的时候，“回文中心”的形式是不一样的。

    - 奇数回文串的“中心”是一个具体的字符，例如：回文串 `"aba"` 的中心是字符 `"b"`；
    - 偶数回文串的“中心”是位于中间的两个字符的“空隙”，例如：回文串串 `"abba"` 的中心是两个 `"b"` 中间的那个“空隙”。

```java
class Solution {
    public String longestPalindrome(String s) {
    if (s == null || s.length() < 1) return "";

    int start = 0, end = 0;
    
    for (int i = 0; i < s.length(); i++) {
        int len1 = expandAroundCenter(s, i, i);
        int len2 = expandAroundCenter(s, i, i + 1);
        int len = Math.max(len1, len2);
        if (len > end - start) {
            start = i - (len - 1) / 2;//前半部分 精确值 偶数长度是（len-1）/2 奇数长度是（len-1）/2
            end = i + len / 2;//后半部分 精确值 偶数长度时是完整的len/2 奇数长度时是（len-1）/2
        }
    }
    return s.substring(start, end + 1);
}

private int expandAroundCenter(String s, int left, int right) {、
//获取以该索引扩散得到的最长回文串长度
    int L = left, R = right;
    while (L >= 0 && R < s.length() && s.charAt(L) == s.charAt(R)) {
        L--;
        R++;
    }
    return R - L - 1;
}

}
```
### [516. 最长回文子序列](https://leetcode-cn.com/problems/longest-palindromic-subsequence/)

- 和最长回文串不一样的地方 这个 子序列 可以不连续 

```java
//leetcode submit region begin(Prohibit modification and deletion)
class Solution {
    public int longestPalindromeSubseq(String s) {
        if(s.length()<2) return s.length();

        char[] ch=s.toCharArray();
        int len=ch.length;
        int[][] dp=new int[len][len];

        for (int i = 0; i < len; i++) {
            dp[i][i]=1;
        }

        for (int i = len-2; i >=0; i--) {
            for (int j = i+1; j < len; j++) {

                if(ch[i]==ch[j]){
                    dp[i][j]=dp[i+1][j-1]+2;
                }else{
                    dp[i][j]=Math.max(dp[i+1][j],dp[i][j-1]);
                }
            }//forj end
        }//fori end
        return dp[0][len-1];
    }
}
//leetcode submit region end(Prohibit modification and deletion)
```
### [72. 编辑距离](https://leetcode-cn.com/problems/edit-distance/)

- 主要是状态的构建 如何去想 

```java

/*
我们采用从未尾开始遍历word1和word2,
当word[i]等于word2[j]时,说明两者完全一样,所以i和j指针可以任何操作都不做,用状
态转移式子表示就是dp[i][j]=dp[i-1][j-1],也就是前一个状态和当前状态是一样的。
当 word1[i]和word2[j]不相等时,就需要对三个操作进行递归了,这里就需要仔细思考状态转
移方程的写法了
对于插入操作,当我们在word1中插入一个和word2一样的字符,那么word2就被匹配了,所以可
以直接表示为dp[i][j-1]+1
对于删除操作,直接表示为dp[i-1][j+1
对于替换操作,直接表示为dp[i-1][j-1]+1
所以状态转移方程可以写成mn(dp[i][j-1]+1,dp[i-1][j]+1,dp[i-1][j-1]+1)
*/
//leetcode submit region begin(Prohibit modification and deletion)
class Solution {
    public int minDistance(String word1, String word2) {
        int n1 = word1.length();
        int n2 = word2.length();
        int[][] dp = new int[n1 + 1][n2 + 1];
        // 第一行
        for (int j = 1; j <= n2; j++) dp[0][j] = dp[0][j - 1] + 1;
        // 第一列
        for (int i = 1; i <= n1; i++) dp[i][0] = dp[i - 1][0] + 1;

        for (int i = 1; i <= n1; i++) {
            for (int j = 1; j <= n2; j++) {
                if (word1.charAt(i - 1) == word2.charAt(j - 1)) dp[i][j] = dp[i - 1][j - 1];
                else dp[i][j] = Math.min(Math.min(dp[i - 1][j - 1], dp[i][j - 1]), dp[i - 1][j]) + 1;
            }
        }
        return dp[n1][n2];
    }
}
//leetcode submit region end(Prohibit modification and deletion)
```

### [198. 打家劫舍](https://leetcode-cn.com/problems/house-robber/)

- 重点是 不相邻 比较简单

```java
class Solution {
    public int rob(int[] nums) {
        if (nums == null || nums.length == 0) {
            return 0;
        }
        int length = nums.length;
        if (length == 1) {
            return nums[0];
        }
        int[] dp = new int[length];
        dp[0] = nums[0];
        dp[1] = Math.max(nums[0], nums[1]);
        for (int i = 2; i < length; i++) {
            dp[i] = Math.max(dp[i - 2] + nums[i], dp[i - 1]);
        }
        return dp[length - 1];
    }
}
```

### [213. 打家劫舍 II](https://leetcode-cn.com/problems/house-robber-ii/)

- 不相邻 问题 加上 首尾不同时 i与0 不同时偷取

```java
class Solution {
    public int rob(int[] nums) {
        if (nums == null || nums.length == 0) {
            return 0;
        }
        if (nums.length == 1) {
            return nums[0];
        }
        return Math.max(robHelp(Arrays.copyOfRange(nums, 1, nums.length)),robHelp(Arrays.copyOfRange(nums, 0, nums.length-1)));
        //主要是在普通版本上排除掉首尾同时偷取的部分，那一开始 我们就分两种情况 再按照不相邻规则偷取 即可
    }
    public int robHelp(int[] nums) {//这部分普通版本搬运过来
        if (nums == null || nums.length == 0) {
            return 0;
        }
        int length = nums.length;
        if (length == 1) {
            return nums[0];
        }
        int[] dp = new int[length];
        dp[0] = nums[0];
        dp[1] = Math.max(nums[0], nums[1]);
        for (int i = 2; i < length; i++) {
            dp[i] = Math.max(dp[i - 2] + nums[i], dp[i - 1]);
        }
        return dp[length - 1];
    }
}
```

**车轮滚滚 时间**

五分钟学算法的动态规划系列:
- [浅谈什么是动态规划以及相关的「股票」算法题](https://mp.weixin.qq.com/s?__biz=MzUyNjQxNjYyMg==&mid=2247485288&idx=1&sn=fd043fc723f38bcaecc90d9945981f8a&chksm=fa0e68e9cd79e1ffd965205bb06b1731539bf2e0bbc5991664f5d1d9721b346ec08c85bb9042&scene=21#wechat_redirect)
- [有了四步解题法模板，再也不害怕动态规划！](https://mp.weixin.qq.com/s?__biz=MzUyNjQxNjYyMg==&mid=2247486904&idx=1&sn=099d5560ab25c0163349dff0c7f51490&chksm=fa0e6239cd79eb2fe6e831d7debba60aa906721d592b8766a944ef88bf91bf82568c20d71891&scene=21#wechat_redirect)
- [（进阶版）有了四步解题法模板，再也不害怕动态规划！](https://mp.weixin.qq.com/s?__biz=MzUyNjQxNjYyMg==&mid=2247486923&idx=2&sn=6c1c8aeb4db68522e67ddf8c1e933660&chksm=fa0e624acd79eb5cdb410808921609a830b9b9221e813e4eb89cf551ca48f317668d44b095d2&scene=21#wechat_redirect)

荐MIT的动态规划练习资料
- [Dynamic Programming Practice Problems](https://people.cs.clemson.edu/~bcdean/dp_practice/)

比较优质的动态规划文章
- [掌握动态规划，助你成为优秀的算法工程师](https://www.jiqizhixin.com/articles/2019-09-29-5)
