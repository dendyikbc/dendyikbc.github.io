---
layout: post
title: '阿里2020/8/28笔试记录'
subtitle: 'JAVA研发笔试'
date: 2020-08-28
author: Dave
cover: ''
tags: 算法与刷题
---

### 真惨

### 1.A->B 最少多少步

记不清完整题干，形如011 到 110 最少需要多少步数，**交换两个元素** 或者 **一次改变一个位数上的（0 1转变）**，或者 **翻转字符串一次**

>上来就觉得像编辑距离，但是这里 多了个翻转 想了想几个思路 不感觉能直接上去dp 可能另有玄机 就先看第二题

事后思路：

不涉及添加 上来就固定长度n

只涉及交换 翻转 01跳变 三类操作

- 想到的 贪心

固定A，对B操作

看两个相似度 这里我想的是用异或 统计相同索引位置不同值的频次K（汉明距离）

**针对每一个操作，向着减小，剩余操作数最大减小的方向传递**

K可能不是最大的减小，但是经过操作后，剩余需要经过的操作数 是减小了最大的

首先 需要操作的索引位置 为 两字符串不一致的位置

然后 每一次 操作的选择 其实会有优先级

**针对翻转**

翻转B再计算相似度 记录K1 并记录剩余所需操作数

剩余所需操作数要比不翻转小2或者更高 才选择翻转 翻转本身会增加一次操作

**针对交换**

如果存在 两个索引位置 i 和 j上元素不同且刚好相反，则进行一次swap(i,j)实现K-2，count+1 

**针对01转变**

对一个索引位置进行reverse(i) 实现K-1，count+1 显然01转变优先级最低

**针对交换和01转变来说，可以合二为一**

统计 0->1个数 和 1->0的个数， 比如有5个 需要从1到0  3个从0到1，那么最少策略为三个交换，剩下的两个01转变 总共所需操作数为5=max{num(0->1),num(1->0)}



然后就变成 每一次去判断 翻转 和 不翻转 之后需要操作数变化，选改变最大那个

比如：

A:1001010

B:0110011

k=5 

count(0->1) = 3

count(1->0) = 2

不翻转情况下，实现两次交换，一次修改即可，需要3次操作，剩余操作数为3

翻转后

 A:1001010

~B:1100110

k=3 

count(0->1) = 2

count(1->0) = 1

翻转情况下，实现一次交换，一次修改即可，需要2次操作，剩余操作数为2 但是本身翻转算一次操作 所以2+1=3 没有变化 可以不翻转

但是 仔细一想 

**整个过程中， 如果需要翻转，那么最多只有一次有效翻转**

>因为这里固定长度 不涉及翻转后的插入，而且是整体翻转，并不支持子段的翻转，所以在这里两次翻转不存在的；

所以 贪心的初衷变成了 简单比较翻转与不翻转的方案 这条道就走通了

```java
/*
 * @ClassName : newcoder
 * @Description : ali828 test
 * @Author : Dave
 * @Date: 2020-08-28 20:04
 **/
import java.util.*;
public class newcoder{
    public static void main(String[] args) {
        Scanner sc = new Scanner(System.in);
        int n = Integer.parseInt(sc.nextLine());
        String A = sc.nextLine();
        String B = sc.nextLine();
        System.out.print(new Solution().solve(n, A, B));
    }
}
class Solution{
    public int solve(int n, String A, String B) {
        return Math.min(getEditNum(A,B) , 1+getEditNum(A,new StringBuilder(B).reverse().toString()));
    }
    public int getEditNum(String A, String B) {
        /*
        * @Param [A, B]
        * @description   //获取字符串A B不翻转情况下最小操作数
        * @author Dave
        * @date 2020/8/28 21:13
        * @return int
        * @throws
        **/
        int n01 = 0, n10 = 0;
        for(int i = 0; i < A.length(); i++) {
            if(A.charAt(i) != B.charAt(i)){
                if(A.charAt(i) =='0'){
                    n01++;
                }else{
                    n10++;
                }
            }//if end
        }//for end
        return Math.max(n01, n10);
    }//method end
}
```

事后想问 笔试时咋想的？不能看着像编辑距离就往上面靠，老想着dp 其实这里 还真不好dp 因为翻转操作的是整个串，如果支持子段翻转，也许dp就可以搞 感觉也不好搞呀dp的起点咋整嘛 万一遇到直接中间翻转就解决问题的那种 真不好处理  

### 2.无重复全排列无前导数 要求能够整除m

>上来就觉得眼熟 全排列的升级版本 感觉dfs怼上去应该就可以了 

>其实 完蛋了比较笨的没有直接计算临时变量,用个队列去表示排列形式，然后在res添加时去判断 结果搞砸了 

```java
/*
不包含前导0  要剪掉  

比如输入 520 的情况下 025就不允许存在  

这一点 可以利用 自然数显示的时候 默认就不包含前导0 代替 所以 对于每一次组合 直接计算出int型的tmp值 
*/
import java.util.*;
public class newcoder {
    public static void main(String[] args) {
        //不包含前导0  要剪掉
        Scanner in = new Scanner(System.in);
        int n = in.nextInt();
        int m = in.nextInt();//这是要去整除的m

        //拆分转成数组
        ArrayList<Integer> list=new ArrayList<>();
        while(n/10!=0){
            list.add(n%10);
            n=n/10;
        }
        list.add(n);
        int[] nn=new int[list.size()];
        for (int i = 0; i < nn.length; i++) {
            nn[i]=list.get(i);
        }
        //todo
        //先排序 再dfs剪枝 去掉前导0和 不能整除m的元素
        Arrays.sort(nn);
        int len=nn.length;
        boolean[] used=new boolean[len];
        List<Integer> res=new LinkedList<>();//符合条件的排列 list
        int tmp=0;//记录临时变量
        dfs(nn,m,0,used,tmp,res);

        System.out.println(res.size());
    }
    public static void dfs(int[] nums,int m,int depth,boolean[] used,int tmp,List<Integer> res) {
        int len=nums.length;
        if(depth==len){
            if(tmp%m==0){
                //todo
                System.out.println(tmp);
                res.add(tmp);
            }
            return;
        }
        for(int i=0;i<len;++i){
            if(used[i]){//使用过的去掉
                continue;
            }
            //剪掉
            if(i>0 && nums[i]==nums[i-1] && !used[i-1]){
                continue;
            }

            tmp=tmp*10+nums[i];
            used[i]=true;

            dfs(nums,m,depth+1,used,tmp,res);

            //回溯回去
            used[i]=false;
            tmp=(tmp-nums[i])/10;
        }//fori end
    }//method end
}

/*TEST
502 2

52
250
502
520
4
*/
```

**车轮滚滚 时间**

- **字符串翻转的实现**

    ```java
    String A = "012345";

    //利用构造方法转StringBuilder再reverse()
    new StringBuilder(str).reverse().toString();


    // 利用toCharArray
    public static String reverse2(String str) {
        char[] chars = str.toCharArray();
        String reverse = "";
        for (int i = chars.length - 1; i >= 0; i--) {
            reverse += chars[i];
        }
        return reverse;
    }

    // 利用charAt
    public static String reverse3(String str) {
        String reverse = "";
        int length = str.length();
        for (int i = 0; i < length; i++) {
            reverse = str.charAt(i) + reverse;
        }
        return reverse;
    }
    ```
